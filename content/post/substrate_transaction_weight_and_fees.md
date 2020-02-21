---
date: 2020-02-21
title: Substrate 区块链应用的交易费用设计
---

[TOC]

通过本文，你将学到：

* 区块链应用为什么存在交易费用
* Substrate 交易费用的组成
* 如何设计更合理的交易费用



## 为什么存在交易费用

在传统物联网（web 2.0）时代，使用微信、微博、淘宝等互联网应用时，终端用户不需要直接费用，而是由服务提供方利用用户的个人信息、产生的数据、注意力等来变现，典型的方式有

* 广告推送；
* 通过用户数据分析指导商家决策；
* 甚至直接共享、贩卖用户隐私等。

在使用以上所说的web 2.0服务时，**用户数据的所有者是服务提供方**。在过去的20年里，虽然我们享受了互联网应用带来的好处，但也时刻“品尝”着隐私泄露、数据主权丢失带来的恶果。

区块链应用将服务的各个组件完全透明化，用户数据的所有权归属于个人，而不是应用的开发者或者任何其他的第三方。用户通过持有私钥掌握着数据，只有持有私钥的人才可以解锁和转移数据，敏感数据往往可以通过加密防止被窃取。

任何区块链应用的出现和流行都离不开这些利益相关方：

* 开发和运营团队
* 维护网络正常运行的节点
* 终端用户

“天下没有免费的晚餐”，终端用户在享受自由的应用服务同时，需要支付相应的服务费用，也就是交易费用，因为服务是由交易触发的。**这些费用可以用来激励相关方更加有效的协作，从而提供更优质的服务**。

**交易费用的另一个目的是在网络和计算资源有限的条件下，高效地调节这些资源的利用率**，而不至于被网络中的垃圾交易所浪费。

在不同的应用场景中，对**资源消耗的成本估算**不尽相同，合理地设计交易费用可以实现参与方的共赢，推动应用的普及。



## 合理设计交易费用

如前文所述，交易费用的目的主要是：

* 激励服务提供方即开发团队和节点
* 调节资源利用率

> 注：本文不考虑通证的通胀和其它的激励措施。

节点和开发团队对交易费用分成，具体的比例由各方根据实际情况协调，并通过链上治理的方式进行动态的调整。

在区块链网络中，典型的资源和相应的费用设计方式如下：

* 有限的区块大小，通过计算每笔交易占用的字节数来收取交易费；
* 有限的区块生成时间，通过计算或者性能测试得出不同交易所消耗的时间；
* 链上状态的存储资源，通常方式有一次性付费和租赁两种模式。一次性付费发生在交易处理过程中，在开发时对此费用评估。租赁模式还会考虑某个交易占据链上状态的时长，超时之后对相应状态进行清除。

## Substrate 交易费用组成

Substrate作为一个通用的区块链应用开发框架，充分考虑了上面提到的各种因素。Substrate设计的交易费用由以下几部分组成：

```
总费用 = 基本费用 +（字节费用 + 权重费用）*（1 + 动态调节费率）+ 小费
```

用于支付的货币由transaction-payment模块的[Currency](https://substrate.dev/rustdocs/master/pallet_transaction_payment/trait.Trait.html#associatedtype.Currency)类型指定，通常由[Balances模块](https://github.com/paritytech/substrate/blob/master/bin/node/runtime/src/lib.rs#L198)给出。

### 基本费用

即[TransactionBaseFee](https://substrate.dev/rustdocs/master/pallet_transaction_payment/trait.Trait.html#associatedtype.TransactionBaseFee)，是每笔交易（特例请参考下面，通过pays_fee设置无付费的交易）都需支付的费用，定义在transaction-payment模块中，在runtime初始化时进行[配置](https://github.com/paritytech/substrate/blob/master/bin/node/runtime/src/lib.rs#L190)，并可以随着runtime的升级进行更新。基本费用的合理设置，可以有效的减少垃圾交易，例如[Kusama网络](https://kusama.network/)的基本费用目前设置为 0.01 ksm。

### 字节费用

在处理区块大小的限制时，Substrate引入了最大区块长度和字节费用，system模块定义了[最大区块长度（MaximumBlockLength）](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.MaximumBlockLength)，transaction-payment模块定义了[每字节的费用（TransactionByteFee）](https://substrate.dev/rustdocs/master/pallet_transaction_payment/trait.Trait.html#associatedtype.TransactionByteFee)，总的字节费用为：

```
字节费用 = 每字节费用 * 字节数
```

和基本费用相同的是，每字节费用也是配置在可升级的runtime代码中。字节数的计算是按照交易的结构体通过[SCALE编码](https://substrate.dev/docs/en/conceptual/core/codec)之后的长度，应用开发者无需过多的关注。以Kusama网络为例，相关的设置如下：

* 最大区块长度：5MB
* 每字节费用：0.0001 ksm

### 权重费用

在有限的区块生成时间和链上状态的限制下，权重被用来定义交易产生的计算复杂度即所消耗的计算资源，以及占据的链上状态。system模块定义了[区块的总权重（MaximumBlockWeight）](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.MaximumBlockWeight)。为了保证在网络繁忙的情况下，依然能够实现对区块链应用有效合理的管理，Substrate引入了两种不同级别的交易类型，既 [Normal 和 Operational](https://substrate.dev/rustdocs/master/frame_support/weights/enum.DispatchClass.html)。Normal类型的交易是由网络中的普通用户提交，Operational类型的交易是由网络中的管理员或者管理委员会共同触发。区块资源如**长度**和**总权重**按照一定比例在这两种类型的交易中进行分配，这一比例称为[可用区块比（AvailableBlockRatio）](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.AvailableBlockRatio)。Kusama网络的设置为：

* 区块的总权重：1,000,000,000
* 可用区块比：75%，即Normal交易最多只占用75%的区块资源，Operational类型的交易则可以占用100%的区块资源，新的交易如果导致对应资源使用率超过阈值后，会被拒绝。

交易（也称为可调用函数）权重设置的方式为：

1. 缺省，使用权重的默认值10,000，参考[代码](https://github.com/paritytech/substrate/blob/master/frame/support/src/weights.rs#L223)。

2. 设置固定权重值和交易级别，[SimpleDispatchInfo](https://substrate.dev/rustdocs/master/frame_support/weights/enum.SimpleDispatchInfo.html)定义了固定权重值的几种方式，

   * FixedNormal，固定权重且为Normal级别的交易
   * InsecureFreeNormal，零权重且为Normal级别的交易
   * FixedOperational，固定权重且为Operational级别的交易

**如何使用固定权重值**，演示代码如下，完整代码请参考[example pallet](https://github.com/paritytech/substrate/blob/master/frame/example/src/lib.rs#L462-L491)：

```rust
// 固定权重的Normal交易
#[weight = SimpleDispatchInfo::FixedNormal(10_000)]
fn accumulate_dummy(origin, increase_by: T::Balance) -> DispatchResult {
    // --snip--
}

// 固定权重的Operational交易
#[weight = SimpleDispatchInfo::FixedOperational(2_000_000)]
fn accumulate_dummy(origin, increase_by: T::Balance) -> DispatchResult {
	// --snip--
}
```

3. 自定义权重计算方法，根据可调用函数的参数进行动态计算，需要一个自定义的结构体，实现`WeighData`、`ClassifyDispatch`和`PaysFee`接口。

   * WeighData：当可调用函数使用某个自定义的权重计算方法时，用来获取该可调用函数的参数列表，并进行相关的计算得出权重。
   * ClassifyDispatch：获取可调用函数的参数列表，合理地判断出不同的交易级别即Normal/Operational。
   * PaysFee: 可以通过设置`pays_fee`为false，来避免收取任何交易费用（小费除外），适用于Operational交易存在权重值，但不收取交易费的场景。

**如何自定义一个权重计算方法**，[example pallet](https://github.com/paritytech/substrate/blob/master/frame/example/src/lib.rs#L502) 中对应的演示代码为：

```rust
// The rules of `WeightForSetDummy` are as follows:
// - The final weight of each dispatch is calculated as the argument of the call multiplied by the
//   parameter given to the `WeightForSetDummy`'s constructor.
// - assigns a dispatch class `operational` if the argument of the call is more than 1000.
struct WeightForSetDummy<T: pallet_balances::Trait>(BalanceOf<T>);

impl<T: pallet_balances::Trait> WeighData<(&BalanceOf<T>,)> for WeightForSetDummy<T>
{
	fn weigh_data(&self, target: (&BalanceOf<T>,)) -> Weight {
		let multiplier = self.0;
		(*target.0 * multiplier).saturated_into::<Weight>()
	}
}

impl<T: pallet_balances::Trait> ClassifyDispatch<(&BalanceOf<T>,)> for WeightForSetDummy<T> {
	fn classify_dispatch(&self, target: (&BalanceOf<T>,)) -> DispatchClass {
		if *target.0 > <BalanceOf<T>>::from(1000u32) {
			DispatchClass::Operational
		} else {
			DispatchClass::Normal
		}
	}
}

impl<T: pallet_balances::Trait> PaysFee<(&BalanceOf<T>,)> for WeightForSetDummy<T> {
	fn pays_fee(&self, _target: (&BalanceOf<T>,)) -> bool {
		true
	}
}

/// A type alias for the balance type from this module's point of view.
type BalanceOf<T> = <T as pallet_balances::Trait>::Balance;
```

**如何使用一个自定义的权重计算方法**：

```rust
#[weight = WeightForSetDummy::<T>(<BalanceOf<T>>::from(100u32))]
fn set_dummy(origin, #[compact] new_value: T::Balance) {
		// --snip--
}}
```

4. 使用Substrate预定义的[`FunctionOf`](https://substrate.dev/rustdocs/master/frame_support/weights/struct.FunctionOf.html)结构体，适用于只有权重需要自定义进行计算，而交易级别固定的情况。FunctionOf接收三个数据，a) 一个根据参数计算权重的closure表达式; b) 固定交易级别或计算交易级别的closure; c) 设置`pays_fee`的布尔值。

**如何使用FunctionOf结构体**：

```rust
// weight = a x 10 + b
#[weight = FunctionOf(|args: (&u32, &u32)| args.0 * 10 + args.1, DispatchClass::Normal, true)]
fn f11(_origin, _a: u32, _eb: u32) { unimplemented!(); }

#[weight = FunctionOf(|_: (&u32, &u32)| 0, DispatchClass::Operational, true)]
fn f12(_origin, _a: u32, _eb: u32) { unimplemented!(); }
```

**注意**：合理的权重值需要通过性能测试来获取，可以参考[PR Weight annotation](https://github.com/paritytech/substrate/pull/3157)；可调用函数的文档中也要明确给出复杂度的计算公式，有多少存储类操作等。

**权重值需要转换为权重费用**，transaction-payment 模块中给出了转换方式的定义[WeightToFee](https://github.com/paritytech/substrate/blob/master/frame/transaction-payment/src/lib.rs#L71)，在runtime模块初始化时给出具体的实现代码，例如在Kusama网路，[WeightToFee的实现](https://github.com/paritytech/polkadot/blob/master/runtime/common/src/impls.rs#L78-L95)为：

```rust
pub struct WeightToFee;

impl Convert<Weight, Balance> for WeightToFee {
    fn convert(x: Weight) -> Balance {
      	// in Polkadot a weight of 10_000 (smallest non-zero weight) to be mapped to 10^7 units of
      	// fees (1/10 CENT), hence:
      	Balance::from(x).saturating_mul(1_000)
    }
}
```



### 动态调节费率

节点的runtime代码中，需要配置`TargetBlockFullness`参数，通常为25%，即在网络平稳运行的过程中，区块资源的使用比例应该稳定在25%左右。当当前区块资源使用超过25%时，将下一区块动态调节费率设置为正，增加交易费用；当资源使用率不足25%时，将下一区块的动态调节费率设置为负，减少交易费用，鼓励交易的发生。这一规则的实现依赖transaction-payment模块的[FeeMultiplierUpdate](https://github.com/paritytech/substrate/blob/master/frame/transaction-payment/src/lib.rs#L74)，Kusama对应的实现代码请参考[这里](https://github.com/paritytech/polkadot/blob/master/runtime/common/src/impls.rs#L97-L149)。



### 小费

西方文化中一个特别之处是，当享用别人提供的优质服务时，会主动给出小费，这种思想也出现在Substrate的设计之中。和现实生活中的小费概念相同，它不是必须的，具体数量由交易发送者任意决定，并且完全由区块生产者获得，而交易费用的其它组成部分会根据一定的比例分配进入“国库”。



## 总结

通过本文，你已经对交易费用有了基本的认识，以及如何合理地使用Substrate提供的交易费用规则。关于交易费用，已经有人在进行一些新的尝试，如：

* 完全没有交易费用，由各个参与方自发组建去中心的网络；
* 对不同的稀缺资源收取一定的租赁费用。



## 引用

[Transaction Weight](https://substrate.dev/docs/en/conceptual/runtime/weight)

[Transaction Fees](https://substrate.dev/docs/en/next/development/module/fees)

[Weights for Pallet Functions](https://hackmd.io/UD0HojfARqyUMC9Jxs5-RA)

[Relay-chain transaction fees and per-block transaction limits](https://research.web3.foundation/en/latest/polkadot/Token%20Economics.html#relay-chain-transaction-fees-and-per-block-transaction-limits)

[Weight annotation](https://github.com/paritytech/substrate/pull/3157)