---
date: 2019-04-11
title: 理解Ethereum智能合约开发
---

[TOC]

## 背景知识

### 为什么是分布式账本？

从第一台计算机的诞生开始，地球的计算能力就一直在按照[摩尔定律](https://zh.wikipedia.org/wiki/%E6%91%A9%E5%B0%94%E5%AE%9A%E5%BE%8B) (每隔两年计算能力翻倍)所指示的那样，持续地在增长（近年有放缓）。**计算早已经不再是一种稀缺的资源**，个人计算机的普及乃至泛滥，也说明了这一现象。

另一方面，我们处在飞速发展的信息社会，各种媒介充斥在生活里的方方面面，**每个人在互联网上留下的一切痕迹，统称为数据，也是信息社会最有价值的**。如果你看过《[西部世界](<https://movie.douban.com/subject/2338055/>)》，你就能够明白数据的力量是有多强大和危险。越来越多的人，意识到这一点，并且不希望自己的数据尤其是私密数据被企业或组织控制。

![data_is_gold](https://github.com/dashengSun/drawio/blob/master/blog/solidity/western_world_start_pic.jpeg?raw=true)

这些企业包含但不局限于支付宝、微信等等独角兽企业。**分布式账本技术去除了对中心化的服务或组织的依赖**，通过提供计算资源，每一个计算单元或者节点（简单的说就是一台计算机）共同维护着一个数据库或者账本，数据库中的数据可以被所有人读，但只有数据的拥有人才能进行转移或者处理，甚至可以进一步通过密码学对数据加密存储，只有持有秘钥的数据拥有人才能读取和处理数据。

![node_connection](https://github.com/dashengSun/drawio/blob/master/blog/solidity/ethereum_peer_network.jpg?raw=true)

**数据的价值在于流动**。不论在什么系统中，交易都是极为重要的，但是交易必须是可控和合理的。任何交易的促成都需要建立在一定的共识基础上。比如中心化的服务支付宝，交易成功的标识是支付宝后台数据库完成价值的转移，并返回成功消息给用户，支付宝是整个交易周期的信任基础。种种原因导致我们并不是百分百地信任支付宝。而去中心化的系统不存在这样的信任中心角色，交易的最终确定是由整个系统基于一定的共识算法完成的，这些算法包括但不局限于工作量证明、权益证明、代理权益证明等。

在分布式账本技术中，我们通过移除信任中心，实现了**数据自由**；通过**开源代码**，建立了另一个维度的信任关系。这项技术，还处在很早期的阶段，需要通过**减缓交易的完成时间来提高安全性**；需要通过**花费大量的计算资源来达到彻底的去中心化**。

### 到底什么是智能合约？

日常生活中充满了各种各样的合约机制，比如房屋租赁合约、银行存取款合约、商务合作合约等等，这些合约的目的都只有一个，就是**保证合约的参与方正常履行自己的义务**。

智能合约的作用也是同样的，只不过合约的形式发生了改变，不再是纸质签名，而是存储在分布式账本中的代码。

![contract_in_code](https://github.com/dashengSun/drawio/blob/master/blog/solidity/contract-in-code.png?raw=true)

为什么要选择智能合约而不是传统合约形式呢？原因我总结大致有以下几个：

* 去中心化的合约形式，**摆脱了对代理人的依赖**，解决了对于代理人的信任问题。
* 分布式账本这个跨越一切国家和地区的技术，使得**自由的全球化经济**更进一步发展，智能合约能够方便快速地满足多种行业全球化经济的需要。
* 智能合约以开源代码的形式展示，使得**合约更加可信、透明**。
* **新的组织、文化、工作形式**，如远程协作办公，开源社团，需要一个更加灵活地协作方式，这里应用智能合约恰到好处。
* 正是由于代码运行在公共的分布式账本上，满足条件的合约可以随时执行，**永不离线**。

合约解决了现实生活中的协作问题，智能合约则将协作的广度和深度推向了另一个高度。

## 以太坊和Solidity

### 以太坊

**分布式账本和智能合约的结合，创造了一种新的协作方式。**以太坊作为一个公共的分布式账本，提供了一个可复制的虚拟机环境，用来执行智能合约代码，保证了每个节点执行相同的代码和交易请求会得到相同的结果。

当应用程序的功能由智能合约来提供时，也常被认为是**去中心化应用** (DAPP)。DApp前端依然是传统的HTML、CSS、JavaScript “三剑客”，通过发送请求到以太坊节点，上传数据到链上，响应链上的事件如智能合约代码的执行结果。对于DApp的后端，从传统中心化的服务器转变为链上可执行的智能合约，通过执行链上的交易请求完成用户账户或者合约账户状态的改变。

越来越多的领域采用DApp的解决方案，例如：

* 知识产权保护
* 金融衍生品交易
* 去中心化的自治组织

### Solidity编程语言

所有上面那些复杂的应用场景，都是用Solidity编写的智能合约来完成的。和大多数高级编程语言类似，Solidity支持面向对象的编程范式，有完善的类型系统，是一种静态编程语言，语法和Javascript很类似。你可以通过[交互式编程环境](https://github.com/raineorshine/solidity-repl)熟悉Solidity的基本用法

#### 基础数据类型

基于按值传递的特性，通常也叫以下类型为[值类型](https://solidity.readthedocs.io/en/v0.5.6/types.html#value-types)。

|                   | 举例                                       | 解释                                            | 操作                                       |
| ----------------- | ------------------------------------------ | ----------------------------------------------- | ------------------------------------------ |
| bool              | true / false                               |                                                 | !, &&, \|\|, ==, !=                        |
| int / uint(8~256) | 8                                          | 默认是256位                                     | 比较，位移，算数加减乘除、取模`**`，位操作 |
| address           | 0x7fD0030D3D21d17Fb4056DE319faD67A853b3C20 | 20字节，也即160位二进制码组成，代表以太坊的地址 | transfer, call                             |
| contract          | contract MyContract {<br />    ...<br />}  | 合约类，是对函数和数据的封装                    | new                                        |
| bytes(1~32)       | "foo"                                      | 定长字节数组                                    |                                            |
| bytes             | "bar"                                      | 变长字节数组                                    |                                            |
| string            | "this is a string"                         | UTF-8字符串                                     |                                            |

#### 枚举类型

例如：

```javascript
enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
```

#### 函数

函数可以是`public`和`internal`的，internal函数只能在合约类内部调用，public函数可以被其他合约类调用。

可以通过`pure`, `view`, `payable`等关键字对函数进行约束：

* view: 函数对于状态可读不可写
* pure: 函数不会与外界状态发生交互，即不会读写状态
* payable: 允许函数接收ether

```javascript
function f(uint num) public pure returns (uint) {
  return num + 1;
}
```

#### 引用类型

由于不同的变量名可以指向同一个引用类型的数据，使用引用类型的时候需要格外小心。常用的引用类型有：

* 数组，如 `[uint(1), 2, 3]`

* 结构体,

  ```javascript
  struct Funder {
    address addr;
    uint amount;
  }
  
  Funder({addr: msg.sender, amount: msg.value});
  ```

* 哈希,

  ```javascript
  mapping(address => uint) public balances;
  
  balances[msg.sender] = newBalance;
  ```

引用类型通过关键字`memory`, `storage`,`calldata`来确定[存储位置](https://solidity.readthedocs.io/en/v0.5.6/types.html#data-location)。

#### 条件控制

即`if else`

```javascript
function max(uint a, uint b) internal pure returns (uint) {
  if (a > b) {
  	return a;
	} else {
  	return b;
	}
}
```

#### 循环控制

即`for, while, break, continue`

```javascript
function sum(uint[] numbers) internal pure returns (uint) {
  uint temp;
  for (i=0; i<numbers.length; i++) {
    temp += numbers[i];
  }
}
```

#### 异常处理

Solidity使用异常来回退状态，常用的方式有`assert, aequire, revert`，抛出的异常不能由代码捕获处理。

```javascript
require(msg.value % 2 == 0, "Even value required.");

assert(address(this).balance == balanceBeforeTransfer - msg.value / 2);

function buy(uint amount) public payable {
  if (amount > msg.value / 2 ether)
    revert("Not enough Ether provided.");
  // Alternative way to do it:
  require(
    amount <= msg.value / 2 ether,
    "Not enough Ether provided."
  );
  // Perform the purchase.
}
```

#### 面向对象特性

**抽象合约类：**内部存在没有实现的方法。

```javascript
contract Feline {
    function utterance() public returns (bytes32);
}
```

**接口：**和其他语言类似，接口内的方法不能有实现。

```javascript
interface Token {
    enum TokenType { Fungible, NonFungible }
    struct Coin { string obverse; string reverse; }
    function transfer(address recipient, uint amount) external;
}
```

**继承和多态：**

```javascript
contract X {}
contract A is X {}
```

**Libraries：** 请参考[这里](https://solidity.readthedocs.io/en/v0.5.6/contracts.html#libraries).

## 常用开发框架

**Truffle:** 基于以太坊平台的开源开发、测试、部署框架，可以一站式完成以太坊智能合约的开发。

快速练习参考[官方资料](https://truffleframework.com/docs/truffle/quickstart)。

常用命令：

```shell
# install
npm install -g truffle

# create project from template
truffle unbox metacoin

# run all the test
truffle test

# compile solidity to abi json file
truffle compile

# start personal ethereum like blockchain
truffle develop

# migrate the contract on blockchain
turffle migrate

# install dependency in ethpm.json
truffle install

# publish your own package
truffle publish

# user debugger
truffle debug <transaction hash>
```

**Ganache:** 本地以太坊私有链客户端，用来本地模拟线上环境，用来本地部署、测试合约。

**Drizzle：**DApp前端开发框架

* 对合约数据响应式编程，包含state,event和transaction
* 声明式，可以减少处理无用数据时的资源浪费。
* 封装底层接口。



## 应用场景练习

### 交易

代码源自`metacoin` template.

ConvertLib.sol:

```javascript
pragma solidity >=0.4.25 <0.6.0;

library ConvertLib{
	function convert(uint amount,uint conversionRate) public pure returns (uint convertedAmount)
	{
		return amount * conversionRate;
	}
}

```

MetaCoin.sol

```javascript
pragma solidity >=0.4.25 <0.6.0;

import "./ConvertLib.sol";

// This is just a simple example of a coin-like contract.
// It is not standards compatible and cannot be expected to talk to other
// coin/token contracts. If you want to create a standards-compliant
// token, see: https://github.com/ConsenSys/Tokens. Cheers!

contract MetaCoin {
	mapping (address => uint) balances;

	event Transfer(address indexed _from, address indexed _to, uint256 _value);

	constructor() public {
		balances[tx.origin] = 10000;
	}

	function sendCoin(address receiver, uint amount) public returns(bool sufficient) {
		if (balances[msg.sender] < amount) return false;
		balances[msg.sender] -= amount;
		balances[receiver] += amount;
		emit Transfer(msg.sender, receiver, amount);
		return true;
	}

	function getBalanceInEth(address addr) public view returns(uint){
		return ConvertLib.convert(getBalance(addr),2);
	}

	function getBalance(address addr) public view returns(uint) {
		return balances[addr];
	}
}

```



## 参考

* [Truffle framework docs](https://truffleframework.com/docs)

