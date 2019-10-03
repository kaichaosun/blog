# 抛硬币游戏(二)：编写测试和UI

在之前的文章 [Substrate应用 - 抛硬币游戏（一）](https://zhuanlan.zhihu.com/p/76815437)，我们完成了runtime的开发，从而实现了一个自定义功能（即抛硬币游戏）的区块链网络。现在让我们来看一下如何编写测试代码和UI，你也可以直接看最终的[模块代码](https://github.com/kaichaosun/substrate-coin-flip/blob/master/runtime/src/mymodule.rs)和[UI代码](https://github.com/kaichaosun/substrate-front-end-template/blob/coinflip/src/CoinFlipGame.js)。

## 测试

### 重要性

为功能模块编写测试，是软件开发过程中不可缺省的一个环节，完备的测试能够：

* 确保代码的执行符合预期；
* 增强重构时的信心；
* 从代码的使用角度，提升代码的设计等。

通常情况下，测试可以分为以下几种：

* 单元测试，mock实现代码中的依赖如其它功能模块，仅测试当前函数的功能；
* 集成测试，不mock实现代码中的依赖，对多个功能模块整体考虑，进行测试；
* End to End 测试，是面向当前系统与依赖的第三方服务之间的测试。

当我们在使用Substrate进行开发时，主要会使用到单元测试和集成测试的方法，对于不同的场景，可以按需选择。一个最佳实践是，**确保自定义的runtime模块有良好的测试覆盖**。

### Rust 测试代码

Rust测试代码通常会和实现代码放在同一个文件或相同的目录下，取决于测试代码的数量，更多内容可以参考[Rust book](https://doc.rust-lang.org/book/ch11-03-test-organization.html)。下面是一个简单的测试用例：

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add_two() {
        assert_eq!(4, add_two(2));
    }
}
```

一些需要注意的点包括：

* 测试代码使用属性`#[cfg(test)]`进行标识
* `use super::*`用来引入当前模块的功能代码
* 测试函数通过属性`#[test]`进行标识
* 断言方式有：
  * 表达式的值为true：`assert!(expression)`
  * 表达式的值是期望的值：`assert_eq!(expected, expression)`
  * 表达式的值不是其它不相关的值：`assert_ne!(other, expression)`
  * 异常发生：`#[should_panic]`
  * Substrate提供了自定义的断言：
    * 结果为`Ok(())`：`assert_ok!(expreesion)`
    * 结果为`Err(error_info)`：`assert_err!(expression, error_info)`
    * 结果为`Err(error_info)，并且不修改存储状态`：`assert_noop!(expression, error_info)`

**运行测试**

```shell
// 运行当前目录下的src目录和tests目录下的所有测试
cargo test

// 运行当前工作空间的所有package下的测试
cargo test --all

// 运行runtime路径下的所有测试，由cargo.toml的`[dependencies.demo-node-runtime]`标识
cargo test -p demo-node-runtime

// 运行runtime路径下单个模块的测试
cargo test -p demo-node-runtime mymodule

// 获取更多帮助信息
cargo test --help
```

运行结果大致如下：

```
running 5 tests
test mymodule::tests::it_works_for_default_value ... ok
test mymodule::tests::play_should_work_for_lose ... ok
test mymodule::tests::set_payment_should_work ... ok
test mymodule::tests::play_should_work_for_win ... ok
test mymodule::tests::play_security_check_should_work ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

### Runtime 测试

为了测试我们的runtime模块，需要首先引入相关的实现代码和依赖，

```rust
use super::*;

use runtime_io::with_externalities;
use primitives::{H256, Blake2Hasher};
// --snip--
```

Runtime模块的功能被封装在一个结构体中，这里我们定义了一个mock的`Test` runtime结构体：

```Rust
pub struct Test;
```

`Test` runtime需要实现被测模块以及所依赖的runtime模块的配置接口，回忆一下我们的模块接口定义，自定义的模块接口继承balances的接口，而 [balances模块又继承了system的接口](https://github.com/paritytech/substrate/blob/master/srml/balances/src/lib.rs#L224)：

```rust
pub trait Trait: balances::Trait {
    type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
}
```

一些不关心的类型如`Event`可以用unit代替，这是因为Substrate为unit类型提供了多数接口的默认实现。这样，接口的实现分别是：

```rust
impl system::Trait for Test {
    type Origin = Origin;
    // --snip--
}

impl balances::Trait for Test {
    type Balance = u64;
  	type OnFreeBalanceZero = ();
    // --snip--
}

impl Trait for Test {
    type Event = ();
}
```

我们还可以对用到的模块定义别名，方便后面的使用：

```rust
type System = system::Module<Test>;
type Balances = balances::Module<Test>;
type mymodule = Module<Test>;
```

还记得runtime模块中callable function的第一个参数是`Origin`枚举类型吗？在实现代码中，`construct_runtime!`宏通过调用`impl_outer_origin! `自动为我们添加了`Origin`的定义。但是在测试代码中，我们需要自己调用`impl_outer_origin!`帮我们生成runtime所依赖的`Origin`类型，宏解析后的相关代码可以参考[这里](https://github.com/kaichaosun/substrate-real-estate-node/blob/master/runtime/expanded.rs#L4039)：

```rust
impl_outer_origin! {
		pub enum Origin for Test {}
}
```

### 初始化GenesisConfig

GenesisConfig存储了链上的原始状态，通常可以用来初始化：

* 预置的账户
* ROOT key
* 账户的余额等。

这里首先初始化了system模块的GenesisConfig为所需的默认值，接着初始化balances模块，将Account ID为1的账户余额设置为10，Account ID为2的账户余额设置为20，其它的如交易费用等设置为0，方便我们计算测试结果。

```rust
fn new_test_ext() -> runtime_io::TestExternalities<Blake2Hasher> {
    let (mut t, mut c) = system::GenesisConfig::<Test>::default().build_storage().unwrap();

    let _ = balances::GenesisConfig::<Test>{
        balances: vec![
            (1, 10),
            (2, 20),
        ],
        transaction_base_fee: 0,
        transaction_byte_fee: 0,
        existential_deposit: 0,
        transfer_fee: 0,
        creation_fee: 0,
        vesting: vec![],
    }.assimilate_storage(&mut t, &mut c).unwrap();

    t.into()
}
```



### 测试set_payment函数

我们通过属性`#[test]`标识了我们的测试函数为`set_payment_should_work`。当payment值为None时，设置payment为参数传入的值，当payment有值时，不做修改。

```rust
#[test]
fn set_payment_should_work() {
    with_externalities(&mut new_test_ext(), || {
        // Set payment when payment is none
        assert_ok!(mymodule::set_payment(Origin::signed(1), 100));
        assert_eq!(mymodule::payment(), Some(100));
        assert_eq!(mymodule::pot(), 100);

        // Set payment when payment is not none
        assert_ok!(mymodule::set_payment(Origin::signed(1), 200));
        assert_eq!(mymodule::payment(), Some(100));
        assert_eq!(mymodule::pot(), 100);
    });
}
```



### 测试play函数

**前置校验条件测试**：

* 函数调用应当是经过签名的
* Payment中存储的值不应为空
* 当用户的余额小于Payment中的值时，应返回错误
* 当用户的余额多于Payment中的值是，应返回正常

```rust
#[test]
fn play_security_check_should_work() {
    with_externalities(&mut new_test_ext(), || {
        // Test ensure_signed
        assert_noop!(mymodule::play(Origin::ROOT), "bad origin: expected to be a signed origin");

        // Test payment must be set
        assert_noop!(mymodule::play(Origin::signed(2)), "Must have payment amount set");

        // Check the balances in genesis config
        assert_eq!(Balances::total_balance(&2), 20);

        // set payment and pot, higher than the balances
        <Payment<Test>>::put(30);
        <Pot<Test>>::put(30);

        assert_noop!(mymodule::play(Origin::signed(2)), "too few free funds in account");

        // set payment and pot, lower than the balances
        <Payment<Test>>::put(10);
        <Pot<Test>>::put(10);
        assert_ok!(mymodule::play(Origin::signed(2)));
    })
}
```

**获胜情景测试**：

* 首先初始化所需的存储状态
* 设定随机种子
* 断言用户获胜后的数据状态

```rust
#[test]
fn play_should_work_for_win() {
    with_externalities(&mut new_test_ext(), || {
        <Payment<Test>>::put(10);
        <Pot<Test>>::put(30);
        <Nonce<Test>>::put(1);
        System::set_random_seed(H256::from_low_u64_be(100));

        assert_ok!(mymodule::play(Origin::signed(2)));
        assert_eq!(mymodule::payment(), Some(10));
        assert_eq!(mymodule::pot(), 10);
        assert_eq!(Balances::total_balance(&2), 40); // 20 - 10 (payment) + 30 (reward)
        assert_eq!(mymodule::nonce(), 2);
    })
}
```

**失败情景测试：**

和获胜情景类似，不过我们选用了不同的随机种子，从而使用户失败:

```rust
#[test]
fn play_should_work_for_lose() {
    with_externalities(&mut new_test_ext(), || {
        <Payment<Test>>::put(10);
        <Pot<Test>>::put(30);
        <Nonce<Test>>::put(1);
        System::set_random_seed(H256::from_low_u64_be(10));

        assert_ok!(mymodule::play(Origin::signed(2)));
        assert_eq!(mymodule::payment(), Some(10));
        assert_eq!(mymodule::pot(), 40);
        assert_eq!(Balances::total_balance(&2), 10);
        assert_eq!(mymodule::nonce(), 2);
    })
}
```

完整的测试代码可参考[这里](https://github.com/kaichaosun/substrate-coin-flip/blob/master/runtime/src/mymodule.rs#L130-L280).

## UI开发

**用户体验差是限制区块链技术应用的一个重要方面**，实用漂亮的UI可以显著的提升用户满意度。这里我们为开发的抛硬币游戏编写一个简单的前端页面。

这里我们使用的是Parity官方的前端应用模板 [substrate-front-end-template](https://github.com/substrate-developer-hub/substrate-front-end-template)，基于

* React,

* Semantic UI

* [polkadot/api](https://github.com/polkadot-js/api)

### 初始化API和状态

根据项目的README文档可以快速启动这一模板应用，默认使用的是本地的节点WebSocket连接，即`ws://127.0.0.1:9944`，配置位于[development.json](https://github.com/substrate-developer-hub/substrate-front-end-template/blob/master/src/config/development.json)。通过模板内置的[useSubstrate](https://github.com/substrate-developer-hub/substrate-front-end-template#usesubstrate-custom-hook)功能函数来创建和Substrate节点交互的辅助组件，如api。

```jsx
const { api } = useSubstrate();
```

新建一个自己的组件`CoinFlipGame.js`，初始化组件所需的state和设置对应state的函数，更多关于`userState`的解释参考 [React官方文档](https://reactjs.org/docs/hooks-state.html)。

```jsx
// The transaction submission status
const [status, setStatus] = useState("");
// The value stored in Payment
const [currentPayment, setCurrentPayment] = useState(0);
// The value stored in Pot
const [potReward, setPotReward] = useState(0);
// The value which will be set to Payment
const [formPayment, setFormPayment] = useState(0);
```

### 状态更新

我们使用[useEffect](https://reactjs.org/docs/hooks-effect.html)当`api.query.mymodule`的值发生变化时，重新设置currentPayment和potReward状态，当需要清理状态时调用相应的unsubscribe函数。

```js
useEffect(() => {
  let paymentUnsubscribe;
  let potUnsubscribe;
  api.query.mymodule.payment(newPayment => {
    // The Payemnt storage value is an Option<Balance>
    // So we have to check whether it is None first
    // There is also unwrapOr
    if (newPayment.isNone) {
      setCurrentPayment('<None>');
    } else {
      setCurrentPayment(newPayment.unwrap().toNumber());
      setStorePaymentDisabled(true);
    }
  }).then(unsub => {
    paymentUnsubscribe = unsub;
  }).catch(console.error);

  api.query.mymodule.pot(reward => {
    setPotReward(reward.toNumber());
  }).then(unsub => {
    potUnsubscribe = unsub;
  }).catch(console.error);

  return () => paymentUnsubscribe && potUnsubscribe &&
    paymentUnsubscribe() && potUnsubscribe();
}, [api.query.mymodule]);
```

### UI组件

显示Payment和Pot的数据组件，如下：

```jsx
<Statistic
  label="Current Payment"
  value={currentPayment}
/>
    
<Statistic
  label="Reward in Pot"
  value={potReward}
/>
```

我们还可以通过输入框设置Payment的初始值，通过Set Payment按钮发送请求到模块的set_payment可调用函数，一旦Payment的值被设置之后，该button的状态就会变成disable。现在，我们可以通过Play按钮玩游戏了，它会发送请求到runtime模块的play函数。这里用到的 [TxButton](https://github.com/substrate-developer-hub/substrate-front-end-template#txbutton-component) 也是模板内置的一个组件，用来帮我们发送query或transaction请求，监控请求的状态。

```jsx
<Form>
  <Form.Field>
    <Input
      type="number"
      id="new_payment"
      label="New Payment"
      onChange={(_, { value }) => setFormPayment(value)}
    />
  </Form.Field>
  <Form.Field>
    <TxButton
      accountPair={accountPair}
      label={'Set Payment'}
      disabled={storePaymentDisabled}
      setStatus={setStatus}
      type='TRANSACTION'
      attrs={{
        params: [formPayment],
        tx: api.tx.mymodule.setPayment
      }}
    />
    <TxButton
      accountPair={accountPair}
      label={'Play Game'}
      setStatus={setStatus}
      type='TRANSACTION'
      attrs={{
        params: [],
        tx: api.tx.mymodule.play
      }}
    />
  </Form.Field>
  <div style={{ overflowWrap: 'break-word' }}>{status}</div>
</Form
```

可调用函数play执行完成后，在Events结果中，可以看到我们定义的`mymodule:PlayResult`事件，及用户的地址和赢得的奖励，测试结果如下图所示。

![coin_flip_play](https://i.imgur.com/GaXEuDy.png)

## 总结

恭喜你，到这里你已经掌握了基于Substrate开发应用链的基本知识：

* 实现runtime逻辑
* 编写测试
* 前端页面 

如果你想学习更多前端开发的知识，请参考[Substrate Front-End Tutorial](https://substrate.dev/docs/en/tutorials/substrate-front-end/)，还可以通过不同的练习如[substrate-collectables-workshop](https://github.com/substrate-developer-hub/substrate-collectables-workshop)，甚至编写自己的业务runtime，来巩固这些知识并熟练应用。

[Substrate开发者中心](https://substrate.dev/)提供了很多学习资料，时常去逛一下，会有不同的收获。



