---
date: 2020-04-16
title: Kusama系列：如何进行链上治理
---

通过本文，你会了解到：

* 典型区块链网络的治理机制有哪些；
* 什么是链上治理；
* Kusama网络的治理机制。


## 为什么需要链上治理

**人类社会的进步，除了依赖技术的创新，还与社会的治理息息相关**。治理体现在社会生活的方方面面，比如：

* 资源的分配，职责的划分；
* 奖励和惩罚机制；
* 未来的治理走向，等等。

治理的表现形式多种多样，有些是明文规定的法律条文，有些是隐性的社会规则。基本上，有人参与的活动就存在治理，小到院子里的小朋友们如何轮流滑滑梯，大到一个企业、国家如果分配输入、划分职责。

合理、透明、高效的治理，能提高社会协作的效率，进而提高**生产效率**和大众的**幸福感**；试想一下，如果你生活在一个没有治理或者治理不合理的环境里，那么社会的协作将会混乱不堪，很难出现高效的生产活动，**公平**和**平等**也就无从谈起。

去中心应用**独特的价值**吸引了越来越多的人来使用，区块链作为去中心应用的一种典型的技术实现方式，拥有着众多的利益相关方，如：

* 区块链应用的终端用户；
* 区块链核心技术开发人员；
* 运营节点的服务商，包括PoW的矿池（矿工）和矿机，PoS的验证人节点等。

那么，已有的区块链系统是如何协调这些利益相关方、实现治理的呢？

比特币由中本聪完成最初版本的开发并上线运行，紧接着第二年，中本聪选择淡出人们的视线，将源代码的控制权移交给社区的开发者。比特币协议的修改需要通过BIP（Bitcoin Improvement Proposals），任何人都可以提交，但是在真正实现某项修改之前，需要充分考虑其安全性和可行性。BIP的生命周期如下图所示，

![bip_life_cycle](/static/governance/bip_life_cycle.jpg)

BIP的成功实施，需要经历这样几个阶段，

* 草案（Draft），提交BIP到开发者邮件列表和Github仓库，收集社区的反馈，讨论、解决反对意见，如果社区内形成了大致共识（rough consensus），就可以进入下一阶段；

  >大致共识是指，对反对的意见进行充分地讨论，直到大多数人认为反对意见是不正确的。

* 提出（Proposed），在这个阶段，BIP拥有了可运行的功能代码，并且制定了部署计划；

* 完成（Final），线上的比特币网络节点部署了这一修改，并且达到了一定的标准，通常需要有全网95%以上的算力触发此项修改。

随着比特币网络的发展，大的数字货币交易所、矿池（矿工）、投资机构在网络的运行上拥有越来越高的话语权，下图展示了当前一周内的算力分布。

![bitcoin_mining_distribution](/static/governance/bitcoin_pool_hashrate.png)

在2017年的[隔离见证（SegWit，BIP141及后续相关的BIP）升级过程](https://bitcoinmagazine.com/articles/long-road-segwit-how-bitcoins-biggest-protocol-upgrade-became-reality)中，各方势力的角逐如同一场没有硝烟的战争，最终分叉成为两个网络。在这一过程中，比特币的治理机制体现地淋漓尽致，既有大的算力集团可以“一手遮天”，也有核心开发者提交多个BIP来应对各种变数，还有持中立态度的部分用户和交易所。很显然，这样的治理**效率低，不够透明，也谈不上真正地去中心**。

另外一个典型的区块链网络是以太坊，它拥有和比特币网络类似的治理机制，通过[EIP（Ethereum Improvement Proposals）](https://eips.ethereum.org/)来管理新特性的实施和部署，不同的是，以太坊的核心开发者在社区拥有极高的声誉和影响力，以太坊基金会的控制力也更强，EIP的实施从流程来看更加清晰，如下图所示。

![eip_workflow](/static/governance/ethereum_governance.jpeg)

从2016年的[The DAO攻击事件](https://www.coindesk.com/understanding-dao-hack-journalists)，可以感受到以太坊网络的治理氛围，在各种社交频道都能看到针对这次攻击及其解决方案的激烈讨论，最终以硬分叉的形式将被黑客盗取的ETH退还给投资人，以太坊基金会及大部分的核心开发者投入到分叉后新的以太坊链，也有少部分的矿工、开发者选择坚守在原有的那条链上。

通过上面两个例子，我们看到了治理在区块链生态里的重要作用，并且有越来越多的团队发现比特币这一类的链下治理（Off-Chain Governance）的不足，并尝试使用链上治理（On-Chain Governance）的方式，来提高治理的效率、透明度，从而进一步实现去中心的目标。[Kusama](https://kusama.network/)就是这样一个采用链上治理的区块链网络。

## Kusama链上治理

治理的核心体现在，当出现意见分歧时，哪一方拥有更高的权力，以及如何做出最终的决定。在Kusama网络里，权力属于KSM token（下面简称为ksm或token）的持有人，最终的决定则由民主投票产生，参与投票的token数量越多、锁定时间越长，权力就越大。Kusama网络的治理概览如下图，

![kusama_governance](/static/governance/kusama_governance.png)

Kusama网络采用了[三院制（Tricameral）](https://en.wikipedia.org/wiki/Tricameralism)的治理结构，

* 公投议院（Referendum chamber，也称为立法院），拥有最广泛的成员（即所有的token持有人）和最高的权利，所有的“立法”（即区块链runtime逻辑的修改）必须经过民主公投；
* 理事会（Council），是Kusama网络中一些日常事务的具体执行机构，其成员由token持有人投票产生；
* 技术委员会（Technical Committee），由开发Polkadot/Kusama网络协议的技术团队组成，作为理事会的补充和制衡，同时受理事会钳制。


民主公投时可使用的投票机制有，

* 绝对多数赞成，公投议案的通过需要获得绝对多数的赞成票，即默认议案不通过；
* 绝对多数反对，只有获得绝对多数的反对票，才能阻止公投议案的通过，即默认议案通过；
* 过半数赞成，公投议案的通过只需要超过一半的投票是赞成票。

其中绝对多数的具体比例和投票率相关，投票率越高，绝对多数所要求的比例越低，比如投票率只有50%时，绝对多数的比例接近80%，当投票率为100%，绝对多数的比例是50%+1。过半数赞成是指无论投票率的高低，赞成票都只需要满足50%+1即可。


### 民主公投

得益于Substrate提供的无分叉升级方式，Kusama网络上任何runtime逻辑的修改，都可以直接通过链上升级来实现。这些修改必须提交议案进行公投，如果公投通过，网络会在一段时间之后自动升级并部署此项修改。[polkadot-js/apps](https://polkadot.js.org/apps/)提供的公投页面如下，
![democracy_page](/static/governance/fast_track.png)

**公众提交议案的流程：**

* 使用Polkadot代码仓库提供的[build-only-wasm脚本](https://github.com/paritytech/polkadot/blob/master/scripts/build-only-wasm.sh)，编译最新的runtime代码，

   ```shell
   scripts/build-only-wasm.sh polkadot // 编译完成后，在根目录生成kusama_runtime.wasm文件
   ```

* 在[polkadot.js/apps Democracy页面](https://polkadot.js.org/apps/#/democracy)（下面简称为某页面），通过`Submit preimage`提交刚生成的wasm文件，并记下对应的哈希值。提交preimage需要质押一定的token，和所提交文件的字节数相关，当提案生效之后，自动归还质押的token。

* 通过`Submit proposal`，填入上一步记录的哈希值和用于锁定的token数量，提交进入公众提案队列。提交议案需要锁定最少1ksm，当针对本议案的公投开始时，锁定解除。别人也可以对提交的议案进行附议（通过`Second`），同样需要锁定token，数量和提交议案时锁定的数量一致。

* Kusama网络每隔7天选择一个新的议案进行公投，这一议案可以是公众提交的支持最高（即提交人和附议人所锁定的token数量最多）的那一个，或者理事会提交的，两种议案轮流进行，如果某一轮其中一类议案为空，则选择另外一类议案。公众提交议案的投票机制总是**绝对多数赞成**。

* 当我们提交的议案进入公投阶段时，就可以在Democray页面的`referenda`下面看到，通过`Vote`对该议案进行投票，投票时，可以给定参与投票的token数量和对议案的信念值（信念值是指如果该议案通过，你希望和新网络“共存亡”的时长，具体的表现是参与投票的token会被锁定的时长，锁定的时间越长，相同数量token的投票权越高）。比如使用10ksm参与投票，信念值是2，那么你的投票权就是20（即10 * 2），假设投的是赞成票，当投票结束后，结果为通过，那么这10个token的锁定时长是从投票结束之后的16天。如果你完全不想锁定，你可以将信念值设为0.1，那你的投票权就只有1（即10 * 0.1）。反对票类似，但是因为投票结果和自己的投票方向相反，从而不会将token进行锁定。

* 公投时长为7天，用户可以在这段时间内的任意时间点进行投票和更改投票。

* 用户可以将自己的投票权（参与投票的token数量和信念值）委托给其它账户，一旦委托，token将被锁定，直到解除委托并且对应的投票锁定时间到期。为了保证大额资金账户的安全，可以设置代理投票账户，从而由代理账户进行常用的投票操作。代理账户也可以将代理的投票权委托出去。

* 投票结束后，关闭公投。对于投票通过的公投，在经过8天的等待时间之后就会自动生效。这一时间超过了用户staking的锁定时长，当用户不满意投票结果时，可以选择不再参与staking，享有完全退出网络的自由。


### 理事会

如果仅仅依赖公投，可以想象治理效率将会很低，所以Kusama网络引入了理事会这样的组织来处理**网络中一些常规事务**，包括但不局限于：

* 取消由于网络异常引发的staking惩罚，需要至少1/2的理事会成员同意；
* 提交非公众的公投议案，这类议案可以有上述3种不同的投票机制，即除了绝对多数赞成，还可以提交过半数赞成和绝对多数反对的议案，前两种需要至少1/2的成员同意，绝对多数反对议案的提交则需要理事会全体成员同意；
* 紧急情况下取消公投，需要2/3的成员同意；
* 对使用国库（treasury）资金的提案进行投票，至少3/5的成员同意才可以通过此类提案，多于1/2的成员则可以直接拒绝。

理事会的成员由持有token的用户投票选举产生的，目前Kusama网络的理事会正式成员有13个，后补7个。选举方式采用的是[Phragmén method](https://wiki.polkadot.network/docs/en/learn-phragmen#council-elections)，每届任期1天，即每24小时重新选举，不过正常情况下成员构成的变化很小，选举流程大致如下：

* 候选人通过Council页面的`Submit candidacy`，来提交候选人申请，需要质押1ksm，如果选举失败没收这1ksm押金，如果成功即成为理事会成员或者后补，则可以把押金取回。
* 通过Council页面上的`Vote`选项，用户可以选择最多16个候选人进行投票，并给出参与投票的token数量，还需要抵押0.05ksm，不过可以随时删除投票，取回押金。
* 选举时间到，结束计票并更新组织成员。

理事会对Kusama网络常规事务的治理是通过提交动议（motion）来实现的，

* 理事会成员通过Council Motions子页面的`Propose montion`选项来提交动议，非理事会正式成员无法提交，提交时需要给出动议所需的最小通过票数（即赞同该动议的最小成员数），并且给出该动议的具体操作，如staking模块用于取消惩罚的`cancelDeferredSlash`操作。为了让动议的具体操作可以成功执行，需要确保动议所需的最少票数满足该操作的要求，如`cancelDeferredSlash`需要至少1/2的成员赞同，即当前13个成员需要有7个投赞成票。
* 投票时间为3天，其它成员针对此动议进行投票，投票通过则立即执行对应的操作，如果时间截止还没有通过，就可以被任何人关闭。但在关闭之前会检查是否存在高级成员（Prime member），如果存在，并且该成员投了赞成票，那么未参与投票的成员会自动跟随该成员也投赞成票，最后进行计算，确定动议是否通过。

理事会还可以提交公投议案（非公众提交的公投议案），这样即使存在很多公众议案的时候，理事会提交的议案每隔一轮总会被取出来进行公投。一个简单的流程如下：

* 在Council Motions子页面，通过`Propose external`提交一个投票规则为过半数赞成的议案，需要提供新的runtime逻辑的代码哈希。
* 理事会其他成员对上述议案进行投票，需要至少1/2的理事会成员同意，才会将该议案放入等待公投的理事会议案队列，该队列目前只能盛放一个议案。
* 在Democracy页面`Fast track`选项，技术委员会的成员可以为该议案申请进入快速通道，如果技术委员会2/3的成员赞同则打开快速通道，这意味着议案可以直接进入公投状态，投票时间缩短为3小时，而如果全体成员赞同则可以取消投票时间的限制。只要成功进入快速通道，不管何种情况，生效时间都没有限制。

### 技术委员会

技术委员会的成员是实现或者定义Polkadot/Kusama协议的团队，实现了其中某一个协议，则占有一个成员席位，如果两个都实现，那么占据两个席位。成员的增减决定需要通过理事会1/2以上的成员同意。技术委员会目前的职责主要包括：

* 上述提供快速通道的功能；
* 否决理事会的公投提案，每个成员针对某个提案只有一次否决机会，并且只能持续7天。

### 国库

Kusama引入了国库的机制，收归国库的费用主要包括，

* 交易手续费分成（80%）；
* staking罚没的金额；
* 理事会候选人落选后的押金；
* 账户被删除后的沉淀资金等。

随着Kusama网络的成长，目前国库的可用余额约为160000ksm，对Polkadot/Kusama生态有益的贡献都可以申请国库内的资金，达到一定程度的激励作用，下面这些领域的贡献都是欢迎的：

* 基础设施的部署和运维；
* 网络安全，如监控服务、审计等；
* 生态支持，比如和第三方区块链的合作；
* 市场活动，包括广告、合作等；
* 社区活动和外联，如见面会，Kusama parties，创客空间等；
* 软件开发类，比如钱包，客户端等等。

申请国库的资金支持有两种方式，提交提案和接受小费（tip）。

**提案**的流程如下：

* 在Riot channel [Kusama Direction](https://matrix.to/#/!QXMnIJzxlnVrvRzhUA:matrix.parity.io?via=matrix.parity.io&via=matrix.org&via=web3.foundation)里，提出申请资金支持的理由，通常需要已经开始了一些工作并且设计了合理的里程碑，听取理事会成员的意见并作相应的改进，得到理事会一定程度的认同后，就可以正式提交提案。通常情况下，充分沟通和得到认同的提案会更容易通过。
* 通过Treasury页面的`Submit proposal`提交申请，需要给出受益人的账户地址和希望得到的资金数额，需要质押申请资金额的5%且不少于20ksm，当申请通过后，质押的token会被返回。
* 每隔6天，自动对通过的提案进行放款。

**小费**的流程为：

* 通过Treasurey页面的`Tip`选项，报告一个值得tip的行为，给出受益人的地址和tip的原因，需要质押1ksm加额外的基于字节数量的费用，当完成tip后，押金可退回。如果tip通过，且报告人和受益人不一致的时候，报告人可以获得tip金额的20%，作为奖励。理事会成员可以直接开启一个tip，无需质押token，且没有报告奖励。
* tip开启之后，理事会成员可以就此进行投票，投票时每个成员给出自己建议的token数量，超过一半的成员赞成就表示通过。需要等1天时间才可以请求发放资金，tip的最终金额是所有建议token数的中间值。


## 总结

通过本文，你已经了解到比特币和以太坊的治理概况，并且掌握了Kusama网络的链上治理模式，公投、理事会、技术委员会，他们互为补充，又彼此制衡。**不管是在区块链系统，还是在现实世界里，没有完美的治理，也无法让所有人满意**。治理和区块链的结合，带来了更大的想象空间，我为此着迷，也希望本文给你带来一些思考。



## 引用

[Kusama Rollout and Governance](https://medium.com/polkadot-network/kusama-rollout-and-governance-31eb18041044)
[Polkadot wiki: Governance](https://wiki.polkadot.network/docs/en/learn-governance)

[The Long Road To SegWit: How Bitcoin’s Biggest Protocol Upgrade Became Reality](https://bitcoinmagazine.com/articles/long-road-segwit-how-bitcoins-biggest-protocol-upgrade-became-reality)

[Bitcoin Governance: What are BIPs and how do they work?](https://blog.sfox.com/bitcoin-governance-what-are-bips-and-how-do-they-work-276cbaebb068)

[EthHub: Governance](https://docs.ethhub.io/ethereum-basics/governance/)

[Understanding The DAO Attack](https://www.coindesk.com/understanding-dao-hack-journalists)

[Polkaassembly governance tool](https://kusama.polkassembly.io/)

## 附录

### democracy pallet

可调用函数:

* propose，提交一个议案，需要锁定至少1ksm，当开启投票之后，解除锁定
* second, 附议某个议案，需要锁定和发起人相同数量的ksm
* vote，给正在进行公投的议案投票
* proxy_vote，代理某个账户进行对公投议案进行投票
* emergency_cancel，紧急情况下取消公投，至少2/3的Council成员同意才可以
* external_propose，1/2的Council成员可以决定下一个由议会提出的公投议案，投票规则为SuperMajorityApprove
* external_propose_majority，1/2的Council成员可以决定下一个由议会提出的公投议案，投票规则为SimpleMajority
* external_propose_default，全体Council成员可以决定下一个由议会提出的公投议案，投票规则为SuperMajorityAgainst（negative-turnout-bias）
* fast_track，2/3的技术委员会成员可以将议会决定的下一个公投议案正式提交进入公投状态，议案的投票规则必须是SimpleMajority或者SuperMajorityAgainst，并缩减投票时间至3小时，投票结束后生效时间没有限制；全体技术委员会成员则可以将投票时间任意缩短。
* veto_external，技术委员会的任何成员可以否决议会决定的下一个公投议案，但是否决的效力只能持续7天。
* cancel_referendum，取消某个公投，只能是root用户
* cancel_queued，取消某个已经完成投票并等待生效的法案，只能是root用户
* on_initialize，每个区块开始的时候执行
  * 如果时间到了（每隔7天），选取一个议案开始进行公投
  * 执行投票通过到达生效期的公投议案
* activate_proxy，stash账户激活某个打开的投票代理账户
* close_proxy，让当前账户不再进行投票代理
* deactivate_proxy，stash账户将激活的代理权收回，使其无效
* delegate，将当前账户的部分投票权委托给另外一个账户，包括投票的token数量和信念值
* undelegate，将当前账户的委托取消
* clear_public_proposals，删除所有的公共提案，只能是root账户
* note_preimage，注册提案的具体操作（即runtime模块的可调用函数），需要锁定一定数量的ksm，和上传提案的可调用函数的字节数长度相关，当提案生效之后，锁定的ksm会解锁。
* note_imminent_preimage，注册紧急提案的具体操作，不需要锁定ksm，需要提案进入到等待生效的阶段（即dispatch queue）。
* reap_preimage，过了投票期（7天）之后，删除注册的操作并解锁注册时锁定的ksm，如果使用的不是注册时的账户，要延长8天。
* unlock，解锁锁定时间过期的ksm
* open_proxy，activate_proxy之前调用，建立代理
* remove_vote，在以下场景下可以删除投票，通常在调用unlock之前
  * 公投还没有结束
  * 公投结束，投票结果输的乙方
  * 公投结束，投票结果为赢的一方，且过了锁定期
* remove_other_vote，替别人删除已经过期的投票
  * 公投被取消
  * 公投结束，投票结果输的乙方
  * 公投结束，投票结果为赢的一方，且过了锁定期

* proxy_delegate，将当前账户代理的stash账户的投票权力委托给另外另外一个账户，包括投票的token数量和信念值
* proxy_undelegate，将当前账户代理的stash账户的委托取消
* proxy_remove_vote，和remove_vote相同，不过是由代理账户触发的
* enact_proposal，将公投正式生效，只能由区块链系统自己触发即root，即scheduler会触发

### collective pallet

可调用函数:

* set_members，设置团体的成员和主席
* execute，以单一成员的身份触发一个可调用函数
* propose，议会成员提交一个motion，并且提交方算一个赞成票
* vote，成员给动议投票，如果通过，马上执行
* close，关闭一个动议，在关闭之前，检查是否通过，如果有prime的话，如果prime是赞成票，那么所有的弃权票相当于赞同票

### treasury pallet

可调用函数：

* propose_spend，国库支出的提案，需要押金，5%的申请金额，最少20ksm
* reject_proposal，多于1/2的理事会成员可以驳回提案，并且没收押金入国库
* approve_proposal，至少3/5的理事会成员可以批准提案，稍后会发放，并且把押金退回
* report_awesome，替自己或者别人申请tip，需要具体的原因，需要锁定至少1ksm和适当的字节费用
* retract_tip，收回tip申请，需要是发起人，成功后解锁抵押的token
* tip_new，主动tip某个行为，不需要锁定token，必须是Tippers即理事会成员。
* tip，对于某个已经开始的tip申请，给出自己建议的token数量，超过一半的Tippers就表示成功，可以关闭tip取现。
* close_tip，关闭某个tip，通过后需要等1天才可以关闭，金额也是现在确定，是所有建议的tipper给出的中间值，且如果报告人和受益人不同，则给报告人20%的分成。
* on_initialize，每隔6天，发放一次国库的钱。

### elections-phragmen pallet

可调用函数：

* vote，给成员、后补、候选人投票，投票的token数量和需要抵押的数量0.05ksm
* remove_voter，删除自己的投票，返回抵押的0.05个ksm
* report_defunct_voter，报告那些非有效的投票人，即他们的投票对象里没有成员、后补和候选人，并获取抵押的0.05 ksm
* submit_candidacy，为自己提交候选人申请，失败了没收1ksm的押金，赢了成为成员或后补，可以把1ksm取回
* renounce_candidacy，放弃候选人身份，取回押金；放弃后补身份，取回押金；放弃成员身份，取回押金，如果后补存在，马上补上
* remove_member，删除某个成员，需要sudo权限，也就是公投

### membership pallet API

可调用函数：

* add_member，新增某成员
* remove_member， 删除某成员
* swap_member，删除某成员，同时新增另一成员
* reset_members，重新设置所有成员
* change_key，将发送交易的账户从成员中删除，并替换为一个新账户
* set_prime，设置成员为高级成员
* clear_prime，删除高级成员