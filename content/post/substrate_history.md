---
date: 2022-01-22
title: Substrate 技术演进
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

大家晚上好，

我是孙凯超，我分享的主题是 substrate 技术演进和生态发展。

![history_framework](/static/substrate/history_framework.png)

有些朋友可能不是很熟悉substrate，我们简单介绍下，

substrate是一个构建区块链的开发框架，可以让开发者在很短的时间的开发一个高可用的区块链应用，而不需要过多的关心一些底层的技术，后面我们会介绍substrate提供了哪些的底层的组件。

通过 substrate 可以让更多的开发者加入进来，使用区块链技术改造自己应用场景，开发对用户更加友好、以用户为中心的应用，

例如保护用户隐私，用户数据所有权等等，同时也就是 下一代互联网 也就是 web3.0 的愿景。



![history_feature](/static/substrate/history_feature.png)

substrate在设计和开发过程中，一个首要的考量是保证软件的可扩展性。

parity工程师考虑了多种可能的使用场景，比如各种类型的公链， 以交易、智能合约为主，或者构建隐私应用的数据存储链，甚至联盟链的一些使用场景。

能做到这一点的一个重要原因是parity有很多区块链客户端的开发经验，比如以太坊，zcash和bitcoin。

substrate的另外一个特点是高度模块化，看过substrate源代码的朋友应该知道，模块的种类和层次是 相当丰富的。

比如数据库有parity db, rocks db，

哈希算法模块有xxhash,blake2, keccak。

公钥加密算法有ed25519, sr25519，ecdsa，开发者可以自由地选择模块加入到自己的应用里。

substrate所有的组件代码都是开源的，已经有超过300名开发者贡献substrate代码库，很多重要的功能是由社区主导开发的，

我们相信，开源是区块链应用的必要条件，同时它也能更好的促进行业发展，推动社会变革。

### Substrate 组件

现在我们来看下substrate提供了哪些区块链开发常用的核心组件，

比如高效的数据库存储层，刚刚提到过的rocksdb,paritydb

网络通信层，substrate使用的是从ipfs抽离出 的libp2p 点对点通信组件库

交易队列或者说交易池，对交易进行一定的排序和验证

还有一个重要的是共识机制，来对多个节点之间的记账操作进行协调，substrate提供的共识机制包括工作量证明 pow，权威证明 proof of authority, 和权益证明 pos。

runtime 模块提供了链上的业务逻辑，用来处理交易，改变链上的状态，典型的业务模块有 资产管理，治理，身份，等等。

上面只列举了一部分功能组件，并且这些组件都可以被方便的扩展，如果现有的组件不满足你的业务要求，还可以自定义  自己需要的组件。


![history_evolvement](/static/substrate/history_evolvement.png)

### Idea

Polkadot 的愿景是一个多链的未来，

那既然需要很多条链，自然需要一个灵活高效的 脚手架。

那在 2018年初，parity 核心开发从原来的polkadot代码库  分离出 substrate，里面包含了 polkadot 用到的所有底层组件。


### Web3 Summit Demo


在2018 年 web3 summit 会议上， Gav 用一台全新的电脑上，演示 使用 substrate在十几分钟内搭建了一个包含客户端、ui的区块链应用，

这个应用是模拟一个简单的抛硬币游戏。

从这以后，substrate 的愿景也从 polkadot 的底层库，转变成 一个通用 、 开源的基于 rust 编程语言 的区块链开发框架

### Evolved with Polkadot

Substrate技术演进的一个主要推动力是polkadot跨链协议的设计，为polkadot提供了一系列的基础功能和核心业务组件，我们之前也提到了这些组件。

substrate不仅要满足 polkadot 的需要，也 一步步添加了各种 区块链开发时所需要的一系列新功能，

例如多种 共识引擎，开发 runtime 所需的API接口，对底层数据库和数据结构 的封装，节点的模板程序等等。

在2019年9月发布了第一个稳定版本 v1.0，可以开始基于这一版本开始构建测试用的区块链应用。

### Enter FRAME


接下来，substrate 发布了 FRAME 这个 模块化的 runtime架构，它简化了区块链业务逻辑的开发，能够将不同的模块进行组合，这些模块通常 我们叫做pallets。

2020年9月发布了功能更加完善的 v2.0 版本，在这段时间里，添加了选多的功能和优化，比如

链下工作机 offchain worker，能够复用链上代码，在链下执行一些计算、IO复杂度比较高的操作，

比如对外部的http请求，对大量的数据的索引和遍历，复杂计算比如 NPoS 这种 质押提名算法。

新增了benchmark 基准测试来确定交易的权重，保证区块链网络中的节点对交易复杂度进行合理的衡量，并且收取一定的交易费用。

通过transactional 的宏保证可调用函数在发生错误时，对链上存储不进行任何修改。

substrate里的 map, double map 这些键值对存储类型引入前缀项，实现快速的遍历和批量删除操作。

新增了40多个功能模块，比如 兼容以太坊的evm 模块，定时执行的scheduling 模块等等

同时，FRAME 代码license 从GPLv3 修改为 apache v2，开发者可以更自由的发布自己的代码。


### FRAME v2

之后，在2021年2月份，Substrate 3.0 发布了新的FRAME v2语法，用新的rust 宏代替了原有的，对代码的可维护性，对 IDE 比如vscode 的支持，都有了比较大的提升。

在3.0里，新增了 对pallets的版本管理 机制，在对链上逻辑进行升级的过程中，数据的迁移也有了更清晰的流程，升级代码所在的工作目录也更加合理。

在许多substrate社区开发者的帮助下，把代码比较多，逻辑复杂的 treasury 模块拆分成了 几个小的模块，比如 bounty，tips, lottery 这几个模块。

另外一个是 mmr ( merkle mountain range )模块，主要是用来辅助区块头 跨链验证的过程，能够 较少计算的开销。

substrate链 之间的跨链操作，以及其他类型的链来验证substrate区块头，都需要用到基于 mmr 模块 的 grandpa 轻客户端。

还有一些重要的修改比如升级了 scale 编解码，优化、重构了 substrate 的网络层。


### v4.0 work in progress

FRAME v2 成熟了，在社区开发者的帮助下，迁移了几十个 功能模块，加快了substrate的迭代速度。

跨链消息的设计， 在 gav 的推动下有了许多重要的更新，它的实现也从 v0 升级到了 v2，有不少社区团队已经在生产环境部署了跨链消息的通道。

还有一些值得注意的新功能，比如

通用化的map数据类型 storage n map，替换了之前的map和double map

bounded vec 是可以在定义的时候 就限制长度的集合类型

counted storage map 记录了 map 里的元素数量，可以用来方便的做一些计算和校验。

开发了新的 scale-info 库，能够自动生成带有类型信息的元数据，客户端比如polkadotjs 或者 subxt 可以根据这个信息，自动地对数据进行 scale 编解码。

删除了 substrate 内置的轻客户端，使用 smoldot 和substrate-connect 来 实现轻客户端的功能，能够在浏览器中直接运行。

还有一个 新增了 transaction-storage 模块，可以用它快速搭建一条 基于 ipfs的存储链。

目前 substrate 4.0 还在开发中，没有发布，不过每个月都会有新的 monthly 小版本出来。

### 2022年更多功能

在新的一年 2022 年，会有更多的功能加进来。

比如把 QUIC 通信协议引入 libp2p和substrate，解决目前节点网络连接的资源消耗大的问题，提升通信的效率。

还打算优化治理模块，不需要过多依赖 council ，实现更加高效的 去中心的治理 模式。

完善smoldot 轻客户端和它的生态，让更多用户在浏览器使用轻客户端访问区块链网络。

bootnodes 查找和发现机制，在节点启动的时候 查找 bootnode, 不需要 hard code 在chain spec 里面。

还有一个不是和代码相关的，也让人很期待的，今年会有一个官方的高标准的教育计划，针对 substrate和polkadot的技术栈。

更多的更新，大家可以去关注parity的代码仓库和社交账户。


我的分享就是这些，

大家可以去官网查看更多资料，还有我们的中文 substrate 技术专栏 

也欢迎大家在 twitter 上跟我交流， id是， kaichaosun

谢谢

