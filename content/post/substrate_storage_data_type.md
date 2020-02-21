---
date: 2020-01-19
title: Substrate存储数据类型概览
---

通过本文，你将学会：

* 区块链应用和传统应用在数据存储层的不同之处；
* 使用区块链进行数据存储时遇到的约束；
* Substrate可用的存储数据类型和使用方法。

如果想更好的理解本文的内容，最好有Substrate runtime的开发经验，你可以根据官方的教程（[Proof Of Existence](https://substrate.dev/docs/en/next/tutorials/creating-your-first-substrate-chain/) 或 [Cryptokitties](https://substrate.dev/substrate-collectables-workshop/#/)）来实践，也可以参考本专栏的其它文章。需要说明的是Substrate源码正在快速迭代更新中，部分语法可能不适用。遇到任何问题欢迎到相应的[渠道](https://subdev.cn/docs/learn_resource/)来咨询。本文源码位于[kaichaosun/play-substrate](https://github.com/kaichaosun/play-substrate)。

## 区块链数据存储的不同

在传统web应用开发领域，数据库相关内容的设计和操作是极为重要的一部分。底层数据库可以分为：

* 关系型数据库，用来存储关系型数据
* 非关系型数据库，可以存储非关系型的多种数据形式，如键值对、半结构化数据等。

以关系型数据为例，在开发过程中通常涉及以下几个方面：

* 数据库选型，常用的有MySQL，PostgreSQL等。
* 设计表结构，需要符合业务的需要并满足一定的原则（也被称为数据库的范式）。
* 编写SQL，或者ORM框架提供的DSL，如Rails提供的Active Record，Java生态里的MyBatis。



区块链作为去中心应用最典型的一种形式，被很多开发者所热衷。区块链应用通常有这样几个特点：

* 发布的代码是**开源可审查**的。
* 运行的程序的是**对等**的，任何人都可以启动并参与到网络中。
* 数据库是去中心的，它**增量地存储数据**，就如同记账一样。
* 通过引入**延迟和随机**来保证账本同一时间只能有一个节点可以写账本，也就是工作量证明（PoW）或者权益证明（PoS）。

**一个已经存在的业务，通过使用区块链的技术、去中心的思想，或许能绽放出新的生命力。**

典型的区块链应用如Bitcoin和Ethereum，它们的客户端软件依赖高效的键值对数据库，比如Bitcoin core 和 Ethereum Go 客户端使用的是LevelDB，Parity Ethereum 和 Substrate 内置的是RocksDB。



## 区块链数据存储的约束

**现实世界的霸权，导致了区块链应用受到大家的追捧**，越来越多的开发者、创业者选择区块链作为自己业务的载体，然而它目前的基础设施还不足以支撑过于复杂的业务场景，以存储为例比如：

* 大文件如图片、视频直接存储在链上的成本很高，我们需要其它的去中心存储方案来解决它。这就导致一个完整的去中心应用可能由多个不同的链来提供服务，而不同链之间的交互也变的不可或缺。
* 链式的区块存储结构不利于对历史数据的索引，如查询某个账户特定交易的所有记录，通常需要一个辅助的链外存储系统来帮助实现高效、自定义的查询功能，以满足终端用户灵活的需求。目前可用的方案只能通过传统中心化的数据库。
* 区块链的共识机制要求所有的节点在运行同一批事务的时候有相同的输出，而浮点数的舍入、计算、比较可能随着不同的编译器、优化程度、计算机架构出现不同的结果，所以区块链应用在进行数值运算时不能使用浮点数。更多内容，请参考[Stop using floating point!](https://bitcointalk.org/index.php?topic=13837.0)，[The trouble with rounding floating point numbers](https://www.theregister.co.uk/2006/08/12/floating_point_approximation/)。



## Substrate 存储单元的数据类型

现在转向今天我们关注的主题，当使用Subsrate开发一条应用链的时候，可以用到哪些存储数据类型和它们相应的操作API。

需要指出的是，存储数据结构的设计需要结合自己的业务进行高度定制。在Substrate的开发过程中不涉及关系表的设计，而是通过它定义的一套标准化接口对数据库中存储的键值对进行增删改查的操作，开发者只需要关注自己的业务，而无需过多地关注与数据库底层的交互，真正地从繁杂的底层开发中解放出来。

Substrate作为一个通用的区块链开发框架，提供了丰富的数据类型用于在链上存储数据。它是基于Rust语言开发的，所支持的数据类型是Rust原生类型的子集（定义在[核心库](https://doc.rust-lang.org/core/index.html)和[alloc库](https://doc.rust-lang.org/alloc/index.html)中），以及这些原生类型构成的映射类型，同时要满足一定的编解码条件。我们通常把它们分为以下四种：

* 单值类型，可用来存储某种单一类型的值，如布尔，数值，枚举，结构体等。
* 简单映射类型，类型标识为map，可以存储键值对，通过key可以索引到value，并进行相应的修改。
* 链接映射类型，类型标识为linked_map，和map类型类似，也是用于存储键值对，不同的是linked_map可以对所有的键值对进行遍历操作，而map目前只能对值（value）进行遍历，不能遍历所有的键（key），更多内容参考这个issue: [Default keys to something enumerable](https://github.com/paritytech/substrate/issues/4610)和之前的PR: [Introduce prefixed storage with enumeration](https://github.com/paritytech/substrate/pull/4185)。
* 双键映射类型，类型标识为double_map，顾名思义，两个key，对应一个value，主要目的是通过第一个键（key 1）快速删除任意key 2的记录，也可以遍历key 1对应的所有的值。

### 单值类型

Rust提供了丰富的基本类型和组合类型，大部分可以在runtime开发中直接使用，并且Substrate还内置了一些独有的类型可以方便地开发去中心应用，部分类型如下表所示：

| 类型                                             | 如何定义                                                     | 默认初始值           | 基本用法                                                     | 说明                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 整数类型，如u8,i8,u32,i32,u64,i64,u128,i128 | `MyUnsignedNumber get(unsigned_number): u8;`                 | 0                    | 增：`MyUnsignedNumber::put(number);`<br />查：`let my_unsigned_num = MyUnsignedNumber::get();`<br />改：`MyUnsignedNumber::mutate(|v| v + 1`<br />删：`MyUnsignedNumber::kill();` | 当数值运算（加减乘除）存在溢出风险时，使用更加安全的api，如返回Result类型的：<br />checked_add,checked_sub,checked_mul,checked_div；一旦溢出返回饱和值的：<br />saturating_add,saturating_sub,saturating_mul 。<br />示例代码：<br />`my_unsigned_num.checked_add(10)?; // if error, fail the transaction`<br />`my_unsigned_num.saturating_add(10086); // => result is 255 for u8` |
| U128,U256,U512                                   | 引入`use primitives::U256;`<br />`MyBigInteger: U256;`       | 0                    | 同u8                                                         | 最新版本为[sp_core::U256](https://substrate.dev/rustdocs/master/sp_core/struct.U256.html) |
| boolean                                          | `MyBool get(my_bool): bool;`                                 | false                | 增删改查基本用法同u8                                         | 取值之后，就可以按照Rust基本语法，通过if else对程序流程进行控制。 |
| char                                             | N/A                                                          | N/A                  | N/A                                                          | 不支持，转为u8再进行存储。                                   |
| string / &str                                    | N/A                                                          | N/A                  | N/A                                                          | 不支持，转为Vec<u8>进行存储，并且对vector的长度应该进行限制，防止对存储空间的过度消耗，更多内容参考 [Substrate wiki - FAQ](https://github.com/paritytech/substrate/wiki/FAQ)。[polkadot-js/common](https://github.com/polkadot-js/common) 提供了工具方法 [stringToU8a](https://github.com/polkadot-js/common/blob/master/packages/util/src/string/toU8a.ts#L39) 从string转换成u8数组。 |
| Vec<T>                                           | 引入：`use rstd::prelude::*;`<br />`MyString get(my_string): Vec<u8>;` | Empty vector         | 增删改查基本用法同u8                                         | 支持Rust vector相关的语法，如push, itererate，参考[Vec 结构体](https://doc.rust-lang.org/alloc/vec/struct.Vec.html)。 |
| float                                            | N/A                                                          | N/A                  | N/A                                                          | 不支持，使用Percent,Permill,Perbill代替，参考前面“区块链存储的约束”一节。 |
| Percent,Permill,Perbill                          | 引入`use sr_primitives::Permill;`<br />`MyPermill get(my_permill): Permill;` | 0                    | 构造Permill, 这里vaule,p,q是u32类型:<br />`let my_permill = Permill::from_percent(value);`, 或<br />`Permill::from_parts(value);`, 或<br />`Permill::from_rational_approximation(p,q);`<br />简单计算：<br />`permill_one.saturating_mul(permill_two);`<br />`my_permill * 20000 as u32;`<br />增删改查基本用法同u8 | 本类型提供了[0,1]之间小数的定点表示方法。实现了一些辅助接口如：<br />saturating_add,saturating_sub,saturating_mul,div,mul，参考[Permill引用文档](https://substrate.dev/rustdocs/master/sp_arithmetic/struct.Permill.html)。 |
| Moment                                           | 当前module的Trait继承`timestamp::Trait`,<br />`pub trait Trait: system::Trait + timestamp::Trait {}`，<br />然后定义：<br />`MyTime get(my_time): T::Moment;` | 0                    | 获取链上的时间：<br /> `<timestamp::Module<T>>::get();`      | timestamp pallet提供了时间类型Moment，通常就是u64的别名，它还提供了获取链上当前时间的接口，但不是实时的，只会在每个区块更新一次，具体更新的细节请参考timestamp模块。 |
| AccountId                                        | `MyAccountId get(my_account_id): T::AccountId;`              | [0u8,32]             | 获取AccountId:<br />`let sender = ensure_signed(origin)?;`   | AccountId通常是`[u8; 32]`的别名。                            |
| H160,H256,H512                                   | 引入：`use primitives::H256;`<br />定义：`MyFixedHash get(my_fixed_hash): H256;` | 以H256为例：[0u8,32] | 增删改查基本用法同u8                                         | 最新版本为[sp_core::H256](https://substrate.dev/rustdocs/master/sp_core/struct.H256.html) |
| Option<T>                                   | `MyOption get(my_option): Option<u32>;` | None | 增删改查基本用法同u8                                         | [Enum core::option::Option](https://doc.rust-lang.org/core/option/enum.Option.html) |
| tuple                                            | `MyTuple get(my_tuple): (u8, bool);`                         | 按需                 | 增删改查基本用法同u8                                         | 取值之后，和Rust语法相同，如：<br />`let (first_elem, second_elem) = MyTuple::get(); // deconstruct value` |
| enum                                             | 参考下面                                                     |                      |                                                              |                                                              |
| struct                                           | 参考下面                                                     |                      |                                                              |                                                              |



**enum枚举**

<u>定义枚举类型</u>:

```rust
#[derive(Copy, Clone, Encode, Decode, Eq, PartialEq, Debug)]
pub enum Weekday {
    Monday,
    Tuesday,
    Wednesday,
    Other,
}

impl Default for Weekday {
    fn default() -> Self {
        Weekday::Monday
    }
}
```

存储的数据需要有默认值，所以需要为自定义的枚举类型实现`Default`接口，并且让Rust编译器自动为我们生成其它所需的接口实现，比如`Copy`、`Clone`、`Eq`，其中`Encode`和`Decode`是Substrate用来编解码存储内容的接口，需要提前引入依赖：`use codec::{Encode, Decode};`。

<u>接着就可以定义自己的枚举存储单元：</u>

```rust
MyEnum get(my_enum): Weekday;
```

<u>增删改查基本用法同u8</u>，示例代码如下：

```rust
// 转换u8为自定义的枚举
impl From<u8> for Weekday {
    fn from(value: u8) -> Self {
        match value {
            1 => Weekday::Monday,
            2 => Weekday::Tuesday,
            3 => Weekday::Wednesday,
            _ => Weekday::Other,
        }
    }
}

// 进行存储
let weekday: Weekday = workday.into();
MyEnum::put(weekday);
```



**struct结构体**

<u>定义我们的结构体</u>，和枚举类似，夜需要实现所需的接口：

```rust
#[derive(Clone, Encode, Decode, Eq, PartialEq, Debug, Default)]
pub struct People {
    name: Vec<u8>,
    age: u8,
}
```

<u>增删改查基本用法同u8</u>，示例代码如下：

```rust
let people = People {
    name: input_name,
    age: input_age,
}
MyStruct::put(value);

let my_people = MyStruct::get();
my_people.name;
my_people.age;
```

**总结**:对于单值类型的存储单元，除了表中列出的一些常用操作如`get,put,mutate,kill`等，Substrate还提供了更多可用的API，具体请参考文档 [Trait frame_support::storage::StorageValue](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html)。如果需要参考更多的示例代码，可以在frame模块下找到。

### 简单映射类型

即map，用来保存键值对，所有上面支持的单值类型都可以用作map中的key或者value。其中，key需要实现`FullEncode`接口，value需要实现`FullCodec`接口，而单值类型实现了`FullCodec`接口，而`FullCodec`继承自`FullEncode`。

<u>定义map：</u>

```rust
MyMap get(my_map): map u8 => Vec<u8>;
```

<u>基本使用方法：</u>

```
// 插入一个元素
MyMap::insert(key, value);

// 通过key获取value
MyMap::get(key);

// 删除某个key对应的元素
MyMap::remove(key);

// 覆盖或者修改某个key对应的元素
MyMap::insert(key, new_value);
MyMap::mutate(key, |old_value| old_value+1);
```

更多可用的API，请参考文档：[frame_support::storage::StorageMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html)。

**需要注意**的是，当使用map来模拟list，也就是将key设置为自增整数时候，直接删除元素，会造成索引的空隙，list长度无限增长，这时可以通过[swap and pop](https://substrate.dev/recipes/storage/enumerated.html#swap-and-pop-for-ordered-lists-a-name--swappopa)的方式来消除这种影响。

### 链接映射类型

即linked_map，也用于存储键值对，拥有map的所有操作，并且可以对所有的键值对进行遍历。

<u>定义linked_map:</u>

```rust
MyLinkedMap get(my_linked_map): linked_map u8 => Vec<u8>;
```

<u>使用基本方法，增删改查和map相同：</u>

```rust
// 遍历键值对
let result: Vec<(u8, Vec<u8>)> = MyLinkedMap::enumerate()
	.filter(|(k, _)| k > &10)
	.collect();
```

通过调用enumerate返回了一个Iterator，然后就可以使用各种帮助方法比如`map,filter,collect,fold`等等。更多内容请参考 [Rust Iterator](https://doc.rust-lang.org/core/iter/trait.Iterator.html) 模式。

### 双键映射类型

即double_map，和map,linked_map不同的是，它使用两个key来索引value，用于快速删除key 1对应的任意记录，也可以遍历key 1对应的所有记录。

<u>定义double_map:</u>

```rust
MyDoubleMap get(my_double_map): double_map T::AccountId, blake2_256(u32) => Vec<u8>; // syntax changed for master
```

<u>使用基本方法：</u>

```rust
let sender = ensure_signed(origin)?; // 获取key1

// 插入一个元素
MyDoubleMap::<T>::insert(sender, key2, value); // key2为参数传入

// 获取某一元素
MyDoubleMap::<T>::get(sender, key2);

// 删除某一元素
MyDoubleMap::<T>::remove(sender, key2);

// 删除key对应的所有元素
MyDoubleMap::<T>::remove_prefix(sender);
```
[Substrate Frame](https://github.com/paritytech/substrate/tree/master/frame)内置的一些模块也使用到了double_map，如：
* [资产管理模块](https://github.com/paritytech/substrate/blob/master/frame/generic-asset/src/lib.rs#L460)中某种资产在账户的余额: AssetId, AccountId => Balance.
* [在线状态模块](https://github.com/paritytech/substrate/blob/master/frame/im-online/src/lib.rs#L241)记录某个Session期间所有验证人生产区块的数量: SessionIndex, ValidatorId => u32.
* [society模块](https://github.com/paritytech/substrate/blob/master/frame/society/src/lib.rs#L449)某个候选人对应的所有投票人的投票结果: Candidate AccountId, Voter AccountId => Vote Result.

遍历key 1对应的记录请参考[StorageDoubleMap API文档](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.iter_prefix)。

### 其它Tips

* 可以通过`pub`关键字设置存储单元的可见范围。
* 可以手动设置默认值，如`MyUnsignedNumber get(unsigned_number): u8 = 10;`。
* 设置初始值的方法，请参考文档 [GenesisConfig Struct](https://substrate.dev/docs/en/1.0/runtime/types/genesisconfig-struct)。
* map,linked_map,double_map的key如果是来自于不可信源，如用户输入，需要使用[加密哈希函数](https://en.wikipedia.org/wiki/Cryptographic_hash_function)（也是Substrate默认使用的）防止数据泄露。
* 不同Substrate版本使用稍有区别，可以在frame目录下查找对应的最新用法。
* [decl_storage](https://substrate.dev/rustdocs/master/frame_support_procedural/macro.decl_storage.html) 宏的说明文档。

  

## 总结

通过本文，你已经基本了解了：

* 使用区块链进行数据存储的不同和约束
* Substrate支持的存储数据类型，以及如何使用不同的类型。

更多内容，请关注知乎专栏：[Substrate区块链开发](https://zhuanlan.zhihu.com/substrate)和网站：[subdev.cn](https://zhuanlan.zhihu.com/substrate)。本文源码位于[kaichaosun/play-substrate](https://github.com/kaichaosun/play-substrate)。



