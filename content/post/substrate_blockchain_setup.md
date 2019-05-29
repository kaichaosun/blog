---
date: 2019-05-28
title: 使用Substrate搭建你的第一条区块链
---

## 为什么使用区块链

比特币网络作为最早的区块链，已经存在了十年时间。在今天，区块链技术还没有像互联网那样深刻地改变着我们的生活，但是它的优势已经在个别行业领域展现出来，比如国际支付、金融衍生品交易、预测市场、去中心化自治组织等。

区块链或者更宽泛的名称是去中心账本，相对于传统互联网行业有其天生的优势：

* 永不离线 - 安全的公有链又全球数以万计的计算机节点共同维护，只要还有一个节点可供访问，数据就不会丢失。
* 开源审查 - 绝大部分区块链应用的代码都是开源的，供全世界的开发人员审查，除了够提升代码质量，开源运动更为深远的意义在于通过分享促进着人类社会的进步。
* 数据加密 - 密码学的运用让数据更加安全。
* 保护隐私 - 用户通过私钥控制着自己的数据。
* 分享权益 - 用户可以通过参与链上的交易活动获取原本只有企业才能拿到的权益。

随着区块链技术的不断演进，交易成本、确认时间、能源消耗、安全性、互通性都得到了极大地提升，传统互联网企业在相同的产业领域将面临着具有以上所说优势的区块链企业的挑战。

作为现在处在优势地位的传统互联网独角兽，如果依靠区块链技术成功转型，在未来不会被轻易淘汰；

作为处于劣势地位的小企业或者小团队，通过在链上实现业务，可以达到"四两拨千斤"的效果。

## 什么是Substrate

[Substrate](https://github.com/paritytech/substrate)是由德国[Parity](https://github.com/paritytech/substrate)公司推出的一个区块链构建框架。它实现了大部分的通用功能，比如点对点网络连接，可配置的共识算法，常用加密算法，数据库存储等。通过使用Substrate，使普通的软件开发人员可以在短时间内建立一条属于自己的完整区块链，开发者只需要关注自己的业务逻辑，从底层复杂的技术解放出来。

使用Substrate构建的区块链，有一个额外的好处，就是可以轻易地连接到Parity的Polkadot公链网络，这一网络具有很多优势，比如跨链交易、共享安全等。

Substrate由Rust语言开发，Rust编程语言具有内存安全、支持静态检查、支持编译为WASM、函数式友好、社区资料完善等优点。通过借助以上Rust的优良特性，导致Substrate的优良性能、可读性高。

官方的参考文档[链接](https://docs.substrate.dev/docs)。

今天我们的主要任务就是使用Substrate来构建一条本地的测试区块链网络。

## 搭建区块链

### 准备环境

* 操作系统是Mac OS 或者Linux的计算机
* Git版本控制工具

### 创建和编译节点程序

#### 方式一

1. 安装依赖工具，如Rust环境、openssl、cmake、 llvm库：

```shell
curl https://getsubstrate.io -sSf | bash
```

如果感兴趣上面脚本的具体执行内容，可以参考[这里](https://getsubstrate.io)。

2. 新建节点程序，使用命令行导航至你想要放置节点程序的目录，执行：

```shell
substrate-node-new substrate-play-node someone
```

 你也可以替换`substrate-play-node`为你想要的节点程序名，替换`someone`为你自己的名字。等待命令执行完后，一个属于自己的节点程序就完成了。

注：`substrate-node-new`命令会帮你拷贝substrate[模板节点程序](https://github.com/paritytech/substrate/tree/master/node-template)、初始化WASM的构建环境、编译Rust代码为WASM等，具体内容参考[链接](https://github.com/paritytech/substrate-up/blob/master/substrate-node-new)。

如果发生了代码改动，那么就需要重新编译。

首先，编译Rust代码为WASM：

```shell
./build.sh
```

之后，编译生成可执行程序：

```shell
cargo build --release
```

可执行程序被放为`target/release/template-node`。

#### 方式二

另外一种创建节点的方式是通过拷贝substrate的Git源码，编译node-template：

```shell
git clone https://github.com/paritytech/substrate

git checkout -b v1.0

cd substrate

cd node-template

# 初始化WASM的构建环境
./scripts/init.sh

# 编译Rust代码为WASM
./scripts/build.sh

# 生成可执行程序
cargo build --release
```

这种方式适合对substrate源码有一定了解的人，*如果是初学者，建议用上面第一种方式*。

### 启动本地单节点开发网络

首先清空节点数据库：

```shell
./target/release/template-node purge-chain --dev
```

然后启动节点程序：

```shell
./target/release/template-node --dev
```

`--dev`表示我们准备启动一个本地开发链，会自动初始化开发环境所需的数据和配置。

如果出现了类似下图所示地内容，那恭喜你成功创建了一条本地开发链。

![dev_node]()

现在你可以访问[https://substrate-ui.parity.io](https://substrate-ui.parity.io)，选择Settings，将remote node设置为Local Node (127.0.0.1:9944)，如下图所示：

![ui_local_node]()

选择Exporer，就能够看到最新的区块在不断产生。

下面，我们来发送一笔交易，选择Extrinsics，`submit the following extrinsic`选择为`balances` `transfer(dest, value)`，`dest: Address`选择为`BOB`，`value: Compact<Balance>`设置为你想转账的金额，之后点击Submit Transaction，几秒钟之后会弹出交易成功的提示，这个时候你可以看到Alice的金额减少了，当切换`using the selected account`为Bob之后，可以看到他的金额增加。

接下来请你要独立地尝试这个页面上的不同功能，不用担心把辛苦搭建的链玩坏，这也是Substrate的一个好处，可以快速清理垃圾数据。

### 启动本地多节点测试网络

启动Alice节点：

```shell
./target/release/template-node \
  --base-path /tmp/alice \
  --chain=local \
  --key //Alice \
  --port 30333 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator \
  --name AlicesNode
```

TODO ![alice_node]()

打开一个新的命令行窗口，启动Bob节点：

```shell
# 替换bootnode的参数值为上面Alice节点的ID
./target/release/template-node \
  --base-path /tmp/bob \
  --chain=local \
  --key //Bob \
  --port 30334 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator \
  --name BobsNode \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/QmbidGG1VJm6to8vm7X4S4J3G5SWJ9ggoxHfuxEYsby43G
```

node-template 命令参数解释：

* `--base-path`
* `--chain`

### 生成新账户用来启动测试网络

生成新账户，包含私钥seed，公钥public key和交易地址Address：

```shell
@ subkey -e generate
Phrase `reject link report slush dinner prison tunnel banner bitter mule bus tennis` is account:
  Seed: 0xde68518018bc509c731a450d18e6c70718458cefd379a3f7e081cd7ce4fe4db5
  Public key (hex): 0x8837f8dd1eeb4300755bfbfc047a26c5a4629c61a71b8ea0a649a8ca9c9e7765
  Address (SS58): 5F9Jzm4jW4LzeAq4V2Cf4QViioagcgyJxMqXdE61TCq1LPtp
```

注：如果没有subkey命令，需要到substrate源码目录下重新编译生成`cargo install --force --path subkey subkey`。

## 参考资料

- [Start a Private Network with Substrate](https://docs.substrate.dev/docs/deploying-a-substrate-node-chain)

