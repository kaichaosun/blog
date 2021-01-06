---
date: 2020-01-06
title: 如何学习substrate源码
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

* 熟练使用substrate的rust文档: crates.parity.io
* 从frame下的感兴趣的pallet开始，比如资产相关的balance, assets; 治理相关的democracy, membership, collective.
  * 模块做什么用，提供了哪些存储和可调用函数
  * 本模块和其它模块如何实现的交互
  * 如何在kusama/polkadot/substrate node/substrate node template的runtime中使用的
  * 理解为什么在链上需要这样的模块或者逻辑
* 对frame的功能有一定了解之后，可以去探索更加底层的知识和架构，比如
  * runtime 模块里对存储单元的操作如何反应在数据库中的
  * 为什么使用wasm，使用的wasm运行时的考量，host和runtime之间的关系和如何互相调用
  * substrate如何使用libp2p实现的点对点网络，及使用的已有协议和新的协议有哪些
  * 共识相关的接口和抽象，可以支持哪些共识，为什么，substrate写新的共识有哪些限制
  * 交易池在substrate架构里的位置以及处理逻辑
  * 通过git issue发现最早添加这个功能的context是什么，有哪些值得关注的讨论
  * more
* 在对底层探索的同时，也不要放弃对业务的理解
  * substrate的那个层次可以支持我的业务，如果要修改我应该去哪里修改
  * 类似的修改是不是通用的，是不是可以贡献回去