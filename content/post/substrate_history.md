---
date: 2022-01-22
title: Substrate 技术演进
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---


![history_framework](/static/substrate/history_framework.png)

有些人可能不是很熟悉 Substrate，我们简单介绍下，

Substrate是一个构建区块链的开发框架，可以让开发者在很短的时间开发一个高可用的区块链应用，而不需要过多的关心一些底层的技术。

通过这个框架可以让更多的应用开发者加入进来，使用区块链技术改造自己应用场景，开发对用户更加友好、以用户为中心的应用，

例如保护用户隐私，用户数据所有权等等，同时也是 下一代互联网（Web 3.0）的愿景。


## Substrate 特性

![history_feature](/static/substrate/history_feature.png)

Substrate在设计和开发过程中，一个首要的考量是保证软件的可扩展性。

Parity的工程师在开发的时候考虑了多种可能的使用场景，比如各种类型的公链，优化交易的UTXO模型，支持WASM智能合约，构建隐私应用的数据存储链等等，甚至还有一些联盟链的使用场景。

能做到这一点的一个重要原因是Parity有很多区块链客户端软件的开发经验，比如以太坊，Zcash和Bitcoin的Rust客户端。

Substrate的另外一个特点是高度模块化，模块的种类和层次相当丰富，建议多看看看相关的源代码，会有更多深入的理解。

我们举几个例子，

* 比如数据库有ParityDB, RocksDB。
* 哈希算法模块有xxhash, blake2, keccak。
* 公钥加密算法有ed25519, sr25519，ecdsa。


开发者可以自由地选择模块加入到自己的应用里。

Substrate所有的组件代码都是开源的，已经有超过300名开发者贡献substrate代码库，很多重要的功能是由社区主导开发的，

我相信，开源是去中心应用的必要条件，同时它也能更好的促进行业发展，推动社会变革。


现在我们来看下substrate提供了哪些区块链开发常用的核心组件，

* 比如刚刚提到过的高效的数据库存储层
* 网络通信层，Substrate使用的是从ipfs抽离出 的libp2p 点对点通信组件库
* 交易队列或者说交易池，对交易进行一定的排序和验证
* 还有一个重要的是共识机制，来对多个节点之间的记账操作进行协调，Substrate提供的共识机制包括工作量证明（POW），权威证明（Proof of Authority）, 和权益证明（PoS）。
* Runtime 模块，它提供了链上的业务逻辑，用来处理交易，改变链上的状态，典型的业务模块有资产管理，治理，身份等等。

上面只列举了一部分功能组件，并且这些组件都可以被方便地扩展，如果现有的组件不满足你的业务要求，还可以自定义自己需要的组件。

## Substrate 演进

![history_evolvement](/static/substrate/history_evolvement.png)

### Idea

提到 Substrate，不得不先说说 Polkadot，它的愿景是一个多链的未来。

那既然需要很多条链，自然需要一个灵活高效的脚手架。

在2018年初，parity 核心开发从原来的Polkadot代码库分离出 Substrate，里面包含了 Polkadot 用到的所有底层组件。


### Web3 Summit Demo

在2018年 Web3 Summit 会议上， Gav用一台全新的电脑上，演示了如何使用Substrate在十几分钟内搭建了一个包含客户端、ui的区块链应用，这个应用是模拟一个简单的抛硬币游戏。

从这以后，Substrate也正式从Polkadot的底层库，转变成一个通用、开源的基于 Rust 编程语言的区块链开发框架。

### Evolved with Polkadot

Substrate技术演进的一个主要推动力是Polkadot跨链协议的设计，为它提供了一系列的基础功能和核心业务组件，我们之前也提到了这些组件。

Substrate不仅要满足Polkadot的需要，同时也一步步添加了各种区块链开发时所需要的一系列新功能，例如

* 多种共识引擎，
* 开发 runtime 所需的API接口，
* 对底层数据库和数据结构的封装，
* 节点的模板程序等等。

在2019年4月发布了第一个稳定版本 v1.0，可以开始基于这一版本开始构建测试用的区块链应用。

### Enter FRAME


接下来，Substrate 发布了 FRAME 这个模块化的runtime架构，它简化了区块链业务逻辑的开发，能够将不同的模块进行组合，这些模块通常我们叫做pallets。

2020年9月发布了功能更加完善的 v2.0 版本，在这段时间里，添加了许多的功能和优化，比如

* 链下工作机 offchain worker，能够复用链上代码，在链下执行一些计算、IO复杂度比较高的操作，比如对外部的http请求，对大量的数据的索引和遍历，复杂计算比如 NPoS 质押提名算法。
* 新增了benchmark基准测试来确定交易的权重，保证区块链网络中的节点对交易复杂度进行合理的衡量，并且收取一定的交易费用。
* 通过 transactional 宏保证交易的原子性，在发生错误时，对链上存储不进行任何修改。
* 对Substrate里的 Map, DoubleMap 这些键值对存储类型引入前缀项，实现快速的遍历和批量删除操作。
* 新增了40多个功能模块，比如兼容以太坊的 evm 模块，定时执行的 scheduling 模块等等。

同时，FRAME代码的 license 从 GPLv3 修改为 Apache v2，开发者可以更自由地发布自己的代码。


### FRAME v2

之后，在2021年2月份，Substrate 3.0 发布了新的FRAME v2语法，用新的 Rust 属性宏代替了原有的过程宏，对代码的可维护性，对IDE比如vscode 的支持，都有了比较大的提升。

在3.0里，新增了对pallets的版本管理机制，在对链上逻辑进行升级的过程中，数据的迁移也有了更清晰的流程，升级代码所在的工作目录也更加合理。

在许多Substrate社区开发者的帮助下，把代码比较多、逻辑复杂的 treasury 模块拆分成了几个小的模块，比如 bounty，tips, lottery 这几个模块。

另外一个是 mmr (Merkle Mountain Range)模块，主要是用来辅助跨链验证区块头的过程，能够较少链上计算的开销。Substrate链之间的跨链操作，以及其他类型的链来验证Substrate链的区块头，都需要用到基于 mmr 模块 的 grandpa 轻客户端。

还有一些重要的修改比如升级了 scale 编解码，优化、重构了 substrate 的网络层。


### v4.0 work in progress

随着 FRAME v2 的成熟，在社区开发者的帮助下，迁移了几十个功能模块，开源协作大大加快了Substrate的迭代速度。

另外一个大的进展是跨链消息的设计，在Gav的推动下有了许多重要的更新，它的实现也从 v0 升级到了 v2，有不少社区团队已经在生产环境部署了跨链消息的通道。

还有一些值得注意的新功能，比如
* 通用化的map数据类型 StorageNMap，替换了之前的Map和DoubleMap。
* BoundedVec 是可以在定义的时候就限制长度的集合类型。
* CountedStorageMap 记录了 map 里的元素数量，可以用来方便的做一些计算和校验。
* 开发了新的 scale-info 库，能够自动生成带有类型信息的元数据，客户端比如 polkadotjs 或者 subxt 可以根据这个信息，自动地对数据进行 scale 编解码。
* 删除了 substrate 内置的轻客户端，使用smoldot和substrate-connect来实现轻客户端的功能，能够在浏览器中直接运行。
* 新增了 transaction-storage 模块，可以用它快速搭建一条基于 ipfs的存储链。

目前 substrate 4.0 还在开发中，没有发布，不过每个月都会有新的 monthly 小版本发布出来。

### 2022年更多功能

在新的一年 2022 年，会有更多的功能加到Substrate的生态中，

* 比如把 QUIC 通信协议引入libp2p和substrate，解决目前节点网络连接的资源消耗大的问题，提升通信的效率。
* 优化治理模块，不需要过多依赖 Council ，实现更加高效的去中心治理模式。
* 完善smoldot轻客户端和它的生态，让更多用户在浏览器使用轻客户端访问区块链网络。
* bootnodes 查找和发现机制，在节点启动的时候查找 bootnode, 不需要硬编码在chainspec文件里。
* 还有一个不是和代码相关的，也让人很期待，今年会有一个官方的高标准的教育计划，针对Substrate和Polkadot的技术栈。

更多更新，大家可以去关注Parity的代码仓库和社交账户。

官网文档: substrate.io 
知乎专栏: parity.link/zhihu

欢迎大家在 twitter 上跟我交流， id是kaichaosun。

