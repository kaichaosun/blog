---
date: 2020-07-20
title: 如何学习Substrate
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

[toc]

通过本文，你能了解到，

* 为什么学习Substrate；
* 学习它所需的必要性知识有哪些；
* Substrate 学习资料和推荐的使用方法；
* 部分学习路径/方法的分享等。

## 为什么学习Substrate

[Substrate](https://github.com/paritytech/substrate)是由Parity公司开发的一个区块链开发框架，提供了区块链开发所需的底层公共组件，可以让普通的开发者快速地开发一条区块链应用，来验证自己的想法。如何你对区块链技术感兴趣，并且想要把区块链的技术带到普通人的生活中，但是没有相关的技术经验，那么，Substrate再适合你不过了。

## 必要性知识

### 区块链知识

如果你对区块链技术能做什么还不太清楚，我建议你把下面这些问题思考一下（不清楚的可以Google搜索），然后和朋友、同事讨论，把讨论结果分享在区块链的爱好者社区里，

* 我们现在的世界存在什么问题？
* 区块链核心的观点/技术点有哪些？
* 区块链能改善或者解决上面提到的问题吗？

如果你想看一下其他人学习区块链的经历，可以参考我之前写的这篇[如何学习区块链技术](https://whisperd.tech/post/how_to_blockchain/)。

对区块链有了初步的了解后，就可以开启我们的Substrate学习之旅了。

### 开发者技能

已经是或者想要成为技术开发者，就必须熟练使用开发者常用到的工具，包括：

* 命令行工具；
* 代码编辑器或者集成开发环境（IDE），如vscode，intellij idea, clion等等；
* Google 搜索，首先用英文搜索（中文只占互联网资料的5%甚至更低）；
* Git/Github 常用操作等。

### 学习Rust

Substrate本身以及基于Substrate的应用链的开发都使用的是Rust语言，一定程度的Rust编程知识是需要的。我建议以下的学习方式：

* 浏览Rust的官方网站https://www.rust-lang.org/，熟悉里面的各种资料，次数不限（我应该看了不下3遍~）。
* 读和练习Rust的官方书籍内容，https://doc.rust-lang.org/book/，定一个固定的时间把它学完（比如一个月），把碎片时间充分利用，这本书的前10章是重中之重。
* 如果学有余力，可以尝试写1~2个自己感兴趣的Rust的小项目。

并不是说要在学习Substrate之前就要把Rust掌握到很高的程度，而是在学习Substrate的过程中，逐步去读上面提到的书和网站提供的各种练手代码等。

一些其它的小工具包括：

* Rust 在线练习环境 https://play.rust-lang.org/，可以快速练习代码片段；
* 如果你熟悉命令行交互执行方式，还可以用 https://github.com/google/evcxr。

## Substrate资料和使用方法

日常获取相关资料的快捷链接：https://subdev.cn/

重要的内容都在 [substrate.dev](https://substrate.dev) 网站上，比如：

* 详细的官方指导文档 https://substrate.dev/docs/en/，比如安装方法、常用的概念和开发指导，首先在整体上熟悉下内容结构，在用到的时候知道在哪里找。一有时间就从前往后读每一节的内容，遇到不理解的可以搜索Google或者先记下跳过，之后在问。
* 不同的教程 https://substrate.dev/en/tutorials，练习的先后顺序建议是：[Create Your First Substrate Chain](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/) -> [Build a PoE Decentralized Application](https://substrate.dev/docs/en/tutorials/build-a-dapp/)，然后是其它的教程。
* 针对单一知识点的代码片段和讲解 https://substrate.dev/recipes/，也是先熟悉内容结构，用到的时候回来查找。
* JS sdk 官方文档[polkadot-js/api](https://polkadot.js.org/api/)，当需要编写前端UI界面时，需要深入学习此文档。

中文的技术资料主要是技术文章和视频，这些内容通常是针对某一个概念或者教程，空闲时间可以多看看：

* Parity中文博客文章: [parity.link/zhihu](https://parity.link/zhihu)
* Parity中文视频: [parity.link/bilibili](https://parity.link/bilibili)

和Substrate核心开发者以及众多的社区技术爱好者互动的渠道有：

* 官方技术支持 Riot 群 [Substrate Technical (Public)](https://matrix.to/#/!HzySYSaIhtyWrwiwEV:matrix.org?via=matrix.parity.io&via=matrix.org&via=web3.foundation)
* 每周的技术研讨会 https://substrate.dev/en/seminar
* 中文Substrate技术交流 Riot 群 [China Substrate Dev](https://matrix.to/#/!trdlqNGrCsZpYUZoXa:matrix.parity.io?via=matrix.parity.io&via=matrix.org&via=web3.foundation)

在这些聊天群里，大家可以提问题，参与讨论，通常会很快获得想要的答案和知识。

另外一些比较重要，但相对深入的资料有[StackOverflow关于Substrate的问答](https://stackoverflow.com/questions/tagged/substrate)、[Substrate的Rust实现文档](https://substrate.dev/rustdocs/)、[源代码](https://github.com/paritytech/substrate)，在用到的时候搜索就可以了。


## 学习路径分享

### 区块链后端（Runtime）开发

这里的后端开发是指的区块链节点代码的开发，在Substrate里也称为runtime开发。

建议的学习路径是：

* 首先尝试这两个教程[Create Your First Substrate Chain](https://substrate.dev/docs/en/tutorials/create-your-first-substrate-chain/) -> [Build a PoE Decentralized Application](https://substrate.dev/docs/en/tutorials/build-a-dapp/)，另外一个[加密猫的教程](https://www.shawntabrizi.com/substrate-collectables-workshop/#/)也可以学习下（版本更新可能不及时）；
* 在练习教程的同时，熟悉substrate.dev和其它中文频道上的各项资料，通过上面提供的资料，了解substrate常用的概念如runtime、宏等；
* 对遇到的问题先尝试搜索已有的资料，如果没有找到，在相应的聊天频道里获取帮助，也可以积极的观察和参与其他人的讨论；
* 完成教程之后，就可以结合自己的兴趣、工作需要，编写自己的感兴趣的区块链功能；

* 如果想部署开发的区块链应用，可以参考博客[Substrate 部署公开测试网络](https://zhuanlan.zhihu.com/p/161293660)。

上面的学习路径并不是固定的，你可以根据自己的情况调整，找到最适合自己的方法。


### 前端开发

一个区块链应用想要被用户接受，必须有一个用户体验良好的前端体验。前端开发者可以这样学习：

* 尝试[Build a PoE Decentralized Application 教程的前端部分](https://substrate.dev/docs/en/tutorials/build-a-dapp/front-end)；
* 了解Substrate的[前端生态](https://substrate.dev/docs/en/knowledgebase/integrate/polkadot-js)和[官方Polkadot-JS 文档](https://polkadot.js.org/api/start/)，对文档中的代码进行练习，熟练掌握前端的开发模式和常用的API；
* 根据自己的区块链runtime逻辑，编写自定义的UI。

## 总结

本文简单介绍了Substrate的一些知识和资料，以及推荐的学习路径和方法，善用这些资料可以辅助大家的学习，在不断练习和阅读里，总结适合自己的方法，希望大家早日开发出自己的区块链应用。