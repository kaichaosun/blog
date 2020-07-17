---
date: 2020-07-17
title: Substrate 部署公开测试网络
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

通过本文，你会了解到：

* 如何修改node template，使它能够提供完整的权益证明（Proof of Stake, PoS）功能；
* Chain Specification的内容，以及如何生成它；
* 配置和启动公开测试网络等。

## 添加 PoS 功能

[Substrate node template](https://github.com/substrate-developer-hub/substrate-node-template) 是一个快速开发Substrate应用链的节点程序，它内置的是权威证明（Proof of Authority, PoA）共识算法，出块算法是 [Aura](https://substrate.dev/docs/en/knowledgebase/advanced/consensus#aura)，也就是出块的节点和顺序是固定不变的。一个线上的公开区块链应用，需要实现一定程度的去中心化，并且通过随机出块来保证安全性，Substrate内置的权益证明（PoS）共识算法就是为了满足此类的需求。

使用 Substrate 添加 PoS 共识算法，通常有两种方法，

1. 使用 [Substrate node](https://github.com/paritytech/substrate/tree/master/bin/node) 节点程序，它内置了几乎所有 Substrate 的功能模块，包括由 [Babe 和 GRANDPA](https://substrate.dev/docs/en/knowledgebase/advanced/consensus#consensus-in-substrate) 组成的 PoS 共识算法；
2. 在 node template 节点程序的基础上，修改或添加所需的功能模块，包括修改出块算法为 BABE，添加用户权益质押（staking）的功能，及治理（governance）相关功能（非必须）等。
3. 也可以使用 [substrate-stencil](https://github.com/kaichaosun/substrate-stencil)，具体方法参考readme文档。

这里我们以第二种即修改 node template 为例，介绍一下具体的流程，这么做的好处是，通过一步步添加新的功能，熟悉各个模块的作用以及它们之间的关系。

### 切换 Babe 

Babe 是Substrate/Polkadot内置的默认出块算法，它的特点是：

* 出块节点由当前的验证人集合随机产生；
* 同时存在次级出块节点，如果当前槽（Slot）没有满足要求的随机区块产生，由次级出块节点生产一个区块；
* 如果长时间不出块比如所有的验证人离线，会导致网络瘫痪，不可恢复。

更多关于Substrate的共识算法，参考官方文档 [Advanced - Consensus](https://substrate.dev/docs/en/knowledgebase/advanced/consensus)。

相应的将 [Aura 切换为 Babe 的代码修改](https://github.com/kaichaosun/tao/commit/610532546d9801f6ba46a6bd325bfbff29629919)如下：

在 runtime/lib.rs 中添加 Babe 模块接口的实现，

```rust
parameter_types! {
    pub const EpochDuration: u64 = EPOCH_DURATION_IN_SLOTS;
    pub const ExpectedBlockTime: u64 = MILLISECS_PER_BLOCK;
}

impl babe::Trait for Runtime {
    type EpochDuration = EpochDuration;
    type ExpectedBlockTime = ExpectedBlockTime;
    type EpochChangeTrigger = babe::SameAuthoritiesForever;
}
```

在 construct_runtime 时引入 Babe 模块，

```rust
Babe: babe::{Module, Call, Storage, Config, Inherent(Timestamp)},
```

实现对应的runtime api，

```rust
impl sp_consensus_babe::BabeApi<Block> for Runtime {
    fn configuration() -> sp_consensus_babe::BabeGenesisConfiguration {
        sp_consensus_babe::BabeGenesisConfiguration {
            slot_duration: Babe::slot_duration(),
            epoch_length: EpochDuration::get(),
            c: PRIMARY_PROBABILITY,
            genesis_authorities: Babe::authorities(),
            randomness: Babe::randomness(),
            allowed_slots: sp_consensus_babe::AllowedSlots::PrimaryAndSecondaryPlainSlots,
        }
    }

    fn authorities() -> Vec<AuraId> {
        Aura::authorities()
    }
    fn current_epoch_start() -> sp_consensus_babe::SlotNumber {
        Babe::current_epoch_start()
    }
}
```

在 Chain spec (chain_spec.rs) 中添加 Babe 的初始区块配置，节点服务启动代码 (service.rs) 修改为 Babe 的启动方式，具体代码请参考[这里](https://github.com/kaichaosun/tao/commit/610532546d9801f6ba46a6bd325bfbff29629919)。

### 添加 staking 功能

Substrate 在共识算法中使用了提名权益证明（Nominated Proof-of-Stake, NPoS），节点可以申请成为验证人，网络中的其它用户可以提名不同的验证人和候选人。每隔一段时间进行选举，使用的选举算法为 [Phragmén method](https://wiki.polkadot.network/docs/en/learn-phragmen)，胜出的节点会成为下一轮正式的验证人，参与区块生产、区块最终性投票等。

NPoS 功能依赖一系列的模块，

* staking：用来管理验证、提名，取消验证、提名，验证人分数统计和奖励发放等；
* session：管理验证人的会话密钥（Session Keys），控制Session的长度和轮换；
* authorship：runtime里用来追踪当前区块生产者和叔块（Uncles），staking模块使用这些信息来统计用于奖励的分数；
* offences, babe, grandpa, im-online：一起协作用来处理验证人的非法行文，如同一个验证人在当前slot生产多个区块，GRANDPA投票人在同一轮针对不同的区块多次投票，大量验证人长时间掉线等；
* utility：实现了批量发送交易的辅助功能，验证人通过前端apps提取奖励时需要用到。

你如果对如何一步步添加以上不同的模块感兴趣，可以参考这里的[代码提交记录](https://github.com/kaichaosun/tao/commits/babe)。

### 添加治理功能

Substrate / Polkadot 的一个创新点在于链上治理，网络中持有 token 的用户可以对链的升级操作进行投票。只有投票通过，才会在一段时间之后更新链上的逻辑即 runtime。相关的模块有：

* treasury：国库，管理网络的公共资金，用户可以通过提案申请资金，也可以由议会成员对用户进行打赏，国库内资金来源有交易费用分成、验证人非法行为的罚没等；
* collective：用来管理一系列账户组成的集体，集体中的成员对动议（通常是可调用函数的调用操作）进行投票，一旦投票通过，以所需的origin执行该可调用函数；
* membership，elections-phragmen：管理某个collective的成员，其中membership通过特定origin进行管理，elections-phragmen通过选举的方式进行管理；
* democracy，scheduler：公投议案的管理如提出、投票、委托，注册原始升级代码等；scheduler模块可以定时执行通过的议案。

也可以查看添加以上模块的[示例代码](https://github.com/kaichaosun/tao/commit/c53f4b4c95a004fc7de185ad74451073120fa7b8)。

## Chain Specification

Chain Spec 包含了一系列配置信息，节点程序用它来连接指定的区块链网络，可连接的引导节点信息，以及初始区块的状态，具体信息如下：

* 元信息如链的name，id；
* 链的类型 chainType，常用的如，Development 启动本地单节点网络，Local 启动本地多节点测试网络， Live 为公开测试网络或正式网络；
* 启动引导节点 bootNodes，即程序启动之后，初始可连接的网络节点有哪些，用来获取区块信息和其它可连接的节点信息；
* 监控服务地址 telemetryEndpoints，节点启动后会定时向该地址发送节点自身的状态如CPU使用率、内存消耗、当前区块信息等等；
* protocolId 用来标识点对点网络传输过程中使用的非libp2p的协议，Substrate 网络传输层依赖了libp2p，除了libp2p规定的协议之外，Substrate 里规定的非libp2p协议需要使用protocolId区分不同的区块链网络，更多内容参考[文档 sc_network](https://crates.parity.io/sc_network/index.html)；
* properties 可包含一些可选的属性信息，如表示token字符的tokenSymbol和小数位数tokenDecimals，前端可以获取相应的此类信息进行展示；
* genesis 即初始区块信息，是所有可用模块的初始信息的集合。

以下是 Chain Spec 的一个示例：

```json
{
  "name": "My Staging Testnet",
  "id": "my_staging",
  "chainType": "Live",
  "bootNodes": [
    "/ip4/127.0.0.1/tcp/30333/p2p/12D3KooWMmsXriTjqeiw4Us9LLgzRGiUmq8f5frBvyJgaYAwNcrU"
  ],
  "telemetryEndpoints": [
    [
      "/dns/telemetry.polkadot.io/tcp/443/x-parity-wss/%2Fsubmit%2F",
      0
    ]
  ],
  "protocolId": "my_staging",
  "properties": {
    "tokenDecimals": 15,
    "tokenSymbol": "MYS"
  },
  "consensusEngine": null,
  "genesis": {
    "runtime": {
      "system": {
        "changesTrieConfig": null,
        "code": "... ..."
      },
      "babe": {
        "authorities": []
      },
      "grandpa": {
        "authorities": []
      },
      "balances": {
        "balances": [
          [
            "5FemZuvaJ7wVy4S49X7Y9mj7FyTR4caQD5mZo2rL7MXQoXMi",
            100000000000000000000
          ],
          ... ...
        ]
      },
      "sudo": {
        "key": "5FemZuvaJ7wVy4S49X7Y9mj7FyTR4caQD5mZo2rL7MXQoXMi"
      },
      "staking": {
        "historyDepth": 84,
        "validatorCount": 8,
        "minimumValidatorCount": 4,
        "invulnerables": [
          "5Grpw9i5vNyF6pbbvw7vA8pC5Vo8GMUbG8zraLMmAn32kTNH",
          ... ...
        ],
        "forceEra": "ForceNone",
        "slashRewardFraction": 100000000,
        "canceledPayout": 0,
        "stakers": [
          [
            "5Grpw9i5vNyF6pbbvw7vA8pC5Vo8GMUbG8zraLMmAn32kTNH",
            "5DLMZF33f61KvPDbJU5c2dPNQZ3jJyptsacpvsDhwNS1wUuU",
            10000000000000000,
            "Validator"
          ],
          ... ...
        ]
      },
      "session": {
        "keys": [
          [
            "5Grpw9i5vNyF6pbbvw7vA8pC5Vo8GMUbG8zraLMmAn32kTNH",
            "5Grpw9i5vNyF6pbbvw7vA8pC5Vo8GMUbG8zraLMmAn32kTNH",
            {
              "babe": "5Dhd2QbrSE4dyNn3YUg8j5TY3fG7ZAWZMoRRF9KUc7VPVGmC",
              "grandpa": "5C6rkxAZB437B5Bf1yS4B4qjW4HZPeBp8Kzx2Se9FLKhfyHY",
              "im_online": "5DscuovXyY1o7DxYroYjYgipn87eqYLyQA3HJ21Utb7TqAai"
            }
          ],
          ... ...
        ]
      },
      "imOnline": {
        "keys": []
      },
      "treasury": {},
      "collectiveInstance1": {
        "phantom": null,
        "members": []
      },
      "collectiveInstance2": {
        "phantom": null,
        "members": [
          "5FemZuvaJ7wVy4S49X7Y9mj7FyTR4caQD5mZo2rL7MXQoXMi",
          ... ...
        ]
      },
      "electionsPhragmen": {
        "members": [
          [
            "5FemZuvaJ7wVy4S49X7Y9mj7FyTR4caQD5mZo2rL7MXQoXMi",
            10000000000000000
          ],
          ... ...
        ]
      },
      "membershipInstance1": {
        "members": [],
        "phantom": null
      },
      "democracy": {}
    }
  }
}
```

上面的例子中，为了易读性缺省了一些必要的信息，如system的code、其他的初始账户和验证人信息等。

那么，如何生成 Chain Spec 呢？

1. 在节点的chain_spec.rs文件中添加自定义的GenesisConfig信息，如初始runtime 的 wasm 代码，初始区块存在的账户和余额，初始验证人的信息，治理团体的信息等：

   ```rust
   fn staging_testnet_genesis() -> GenesisConfig {
       // subkey inspect "$SECRET"
       let endowed_accounts = vec![
           // 5FemZuvaJ7wVy4S49X7Y9mj7FyTR4caQD5mZo2rL7MXQoXMi
           hex!["9eaf896d76b55e04616ff1e1dce7fc5e4a417967c17264728b3fd8fee3b12f3c"].into(),
           ... ...
       ];
   
       // for i in 1 2 3 4; do for j in stash controller; do subkey inspect "$SECRET//$i//$j"; done; done
       // for i in 1 2 3 4; do for j in babe; do subkey --sr25519 inspect "$SECRET//$i//$j"; done; done
       // for i in 1 2 3 4; do for j in grandpa; do subkey --ed25519 inspect "$SECRET//$i//$j"; done; done
       // for i in 1 2 3 4; do for j in im_online; do subkey --sr25519 inspect "$SECRET//$i//$j"; done; done
       let initial_authorities: Vec<(
         AccountId,
         AccountId,
         BabeId,
         GrandpaId,
         ImOnlineId,
       )> = vec![(
         // 5Grpw9i5vNyF6pbbvw7vA8pC5Vo8GMUbG8zraLMmAn32kTNH
         hex!["d41e0bf1d76de368bdb91896b0d02d758950969ea795b1e7154343ee210de649"].into(),
         // 5DLMZF33f61KvPDbJU5c2dPNQZ3jJyptsacpvsDhwNS1wUuU
         hex!["382bd29103cf3af5f7c032bbedccfb3144fe672ca2c606147974bc2984ca2b14"].into(),
         // 5Dhd2QbrSE4dyNn3YUg8j5TY3fG7ZAWZMoRRF9KUc7VPVGmC
         hex!["48640c12bc1b351cf4b051ac1cf7b5740765d02e34989d0a9dd935ce054ebb21"].unchecked_into(),
         // 5C6rkxAZB437B5Bf1yS4B4qjW4HZPeBp8Kzx2Se9FLKhfyHY
         hex!["01a474a93a0cf830fb40b1d17fd1fc7c6b4a95fa11f90345558574a72da0d4b1"].unchecked_into(),
         // 5DscuovXyY1o7DxYroYjYgipn87eqYLyQA3HJ21Utb7TqAai
         hex!["50041e469c63c994374a2829b0b0829213abd53be5113e751043318a9d7c0757"].unchecked_into(),
       ),
       ... ...
       ];
   
       const ENDOWMENT: u128 = 1_000_000 * DOLLARS;
       const STASH: u128 = 100 * DOLLARS;
       let num_endowed_accounts = endowed_accounts.len();
   
       GenesisConfig {
         system: Some(SystemConfig {
           code: WASM_BINARY.to_vec(),
           changes_trie_config: Default::default(),
         }),
         balances: Some(BalancesConfig {
           balances: endowed_accounts.iter()
             .map(|k: &AccountId| (k.clone(), ENDOWMENT))
             .chain(initial_authorities.iter().map(|x| (x.0.clone(), STASH)))
             .collect(),
         }),
         babe: Some(BabeConfig {
           authorities: vec![],
         }),
         grandpa: Some(GrandpaConfig {
           authorities: vec![],
         }),
         sudo: Some(SudoConfig {
           key: endowed_accounts[0].clone(),
         }),
         session: Some(SessionConfig {
           keys: initial_authorities.iter().map(|x| {
             (
               x.0.clone(),
               x.0.clone(),
               session_keys(x.2.clone(), x.3.clone(), x.4.clone())
             )
           }).collect::<Vec<_>>(),
         }),
         staking: Some(StakingConfig {
           validator_count: initial_authorities.len() as u32 * 2,
           minimum_validator_count: initial_authorities.len() as u32,
           stakers: initial_authorities
             .iter()
             .map(|x| (x.0.clone(), x.1.clone(), STASH, StakerStatus::Validator))
             .collect(),
           invulnerables: initial_authorities.iter().map(|x| x.0.clone()).collect(),
           force_era: Forcing::ForceNone,
           slash_reward_fraction: Perbill::from_percent(10),
           .. Default::default()
         }),
         im_online: Some(ImOnlineConfig {
           keys: vec![],
         }),
         democracy: Some(DemocracyConfig::default()),
         elections_phragmen: Some(ElectionsConfig {
           members: endowed_accounts.iter()
                 .take((num_endowed_accounts + 1) / 2)
                 .cloned()
                 .map(|member| (member, STASH))
                 .collect(),
         }),
         collective_Instance1: Some(CouncilConfig::default()),
         collective_Instance2: Some(TechnicalCommitteeConfig {
           members: endowed_accounts.iter()
                 .take((num_endowed_accounts + 1) / 2)
                 .cloned()
                 .collect(),
           phantom: Default::default(),
         }),
         membership_Instance1: Some(Default::default()),
         treasury: Some(Default::default()),
       }
   }
   ```

   初始区块的账户可以是网络的管理员、发起者，或者其他社区成员，初始余额可以根据对网络的贡献进行分配。我们可以使用 subkey 工具生成所需的初始验证人信息，如stash和controller 账户，以及由Babe、GRANDPA、im-online组成的Session Keys，其中GRANDPA使用的是ed25519加密算法，其它则使用sr25519加密算法。subkey的使用方法可以参考[官方文档 The subkey Tool](https://substrate.dev/docs/en/knowledgebase/integrate/subkey)，这里用到的命令在代码的注释中已经给出。

   接着，就可以使用 `ChainSpec::from_genesis` 将通用的配置信息和 GenesisConfig 组合成Chain Spec，这里bootnodes的信息可以为空，在我们启动bootnode并获取到它的PeerId（打印在命令行输出）之后，修改Chain Spec 的 JSON 文件即可：

   ```rust
   pub fn staging_testnet_config() -> ChainSpec {
       let boot_nodes = vec![];
   
       ChainSpec::from_genesis(
         "My Staging Testnet",
         "my_staging",
         ChainType::Live,
         staging_testnet_genesis,
         boot_nodes,
         Some(
           TelemetryEndpoints::new(vec![("wss://telemetry.polkadot.io/submit/".to_string(), 0)])
             .expect("Westend Staging telemetry url is valid; qed")
         ),
         Some("my_staging"),
         None,
         Default::default(),
       )
   }
   ```

2. 修改 command.rs 里的 load_spec，添加公开测试网络的解析方式：

   ```rust
   fn load_spec(&self, id: &str) -> Result<Box<dyn sc_service::ChainSpec>, String> {
   		Ok(match id {
           "dev" => Box::new(chain_spec::development_config()),
           "" | "local" => Box::new(chain_spec::local_testnet_config()),
           "my-staging" => Box::new(chain_spec::staging_testnet_config()),
           path => Box::new(chain_spec::ChainSpec::from_json_file(
             std::path::PathBuf::from(path),
           )?),
   		})
   }
   ```

3. 使用build-spec子命令生成Chain Spec 配置文件，命令为`./target/release/node-template build-spec --chain my-staging > my-staging.json`；

4. 如果要发布公开网络，需要使用编码后的Chain Spec文件，从而确保网络中的各个节点使用相同的初始状态即genesis hash相同，命令为：`./target/release/node-template build-spec --chain=my-staging.json --raw > my-staging-raw.json`，将此文件分发即可，也可以修改 staging_testnet_config，添加以配置文件加载网络的解析方式，

   ```rust
   /// Staging testnet generator
   pub fn staging_testnet_config() -> Result<ChainSpec, String> {
       ChainSpec::from_json_bytes(&include_bytes!("../res/my-staging-raw.json")[..])
   }
   ```

5. 配置bootnodes，启动某个bootnode的命令为：

   ```bash
   ./target/release/node-template \
       --node-key c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a \
       --base-path /tmp/bootnode1 \
       --chain my-staging-raw.json \
       --name bootnode1
   ```

   node-key 可使用 subkey ed25519的方式生成；如果默认的chain就是my-staging-raw.json所指定的，`--chain`也可以缺省。

   记录打印的PeerId（即这里的`12D3KooWMmsXriTjqeiw4Us9LLgzRGiUmq8f5frBvyJgaYAwNcrU`），结合bootnode的IP地址或者域名更新 Chain Spec 文件并重新分发。最好有多个bootnodes，防止出现单点故障。

   ```json
   {
     "name": "My Staging Testnet",
     "id": "my_staging",
     "chainType": "Live",
     "bootNodes": [
       "/ip4/your-ip-address/tcp/30333/p2p/12D3KooWMmsXriTjqeiw4Us9LLgzRGiUmq8f5frBvyJgaYAwNcrU"
     ],
     ... ...
   }
   ```



## 启动公开测试网络

为了确保公开网络的发布过程顺利，不会出现大范围的故障，这里我们借鉴 Kusama/Polkadot 正式网络的发布流程，具体可以分为下面几个步骤：

![launch_process](/static/launch/launch_process.png)

在部署和升级的过程中，可以借助一些监控和辅助工具如：

* Prometheus，用来监控节点的状态，[参考文档 Visualizing Node Metrics](https://substrate.dev/docs/en/tutorials/visualize-node-metrics/)；
* [srtool](https://gitlab.com/chevdor/srtool)，编译 runtime 的 wasm 代码，用来解决不同的硬件、操作系统导致同样的代码编译结果不同，无法验证的问题；
* 其它[工具](https://wiki.polkadot.network/docs/en/build-tools-index)。

### 启动网络的 PoA 模式

在这一阶段，要确保：

* Staking模块的初始配置即 StakingConfig 的 force_era 设置为 **ForceNone**，这样的话，网络的验证人不会变化；

* 使用 system 模块提供的 `BaseCallFilter` 过滤掉当前阶段非必须的功能模块，从而不会因为网络不稳定导致无效交易的发生，示例代码参考[这里](https://github.com/kaichaosun/tao/blob/babe/runtime/src/lib.rs#L255)；

* 启动初始验证人，命令为：

  ```bash
  ./target/release/node-template \
      --base-path  /tmp/validator1 \
      --chain   my-staging-raw.json \
      --bootnodes  /ip4/your-ip/tcp/30333/p2p/12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2 \
      --name  validator1 \
      --validator
  ```

  `--chain`和`--bootnodes`如果已经配置在默认启动模式下，可缺省；需要将初始验证人集合所指定的验证人都启动，并且不少于3个。

  启动之后，给每个验证人设置 Session Keys，这里以curl发送http请求进行添加为例，

  ```bash
  // 添加Babe的密钥
  curl http://localhost:9933  -H "Content-Type:application/json;charset=utf-8" -d "@babe1"
  
  // babe1文件的内容
  {
      "jsonrpc":"2.0",
      "id":1,
      "method":"author_insertKey",
      "params": [
          "babe",
          "own word vocal dog decline set bitter example forget excite gesture water//1//babe",
          "0x48640c12bc1b351cf4b051ac1cf7b5740765d02e34989d0a9dd935ce054ebb21"
      ]
  }
  ```

  GRANDPA，im-online 密钥的设置方式类似，更多内容可以参考文档 [Creating Your Private Network](https://substrate.dev/docs/en/tutorials/start-a-private-network/customchain#add-keys-to-keystore)。注意：**GRANDPA密钥设置之后，需要重启节点**。

* 允许提名和验证意向，等待拥有充足的提名和验证人之后，就可以考虑切换为 PoS 模式。

### 切换网络为 PoS 模式

网络在PoA阶段稳定并且没有可见的故障后，可以使用 sudo 权限调用 staking 模块的 force_new_era，开启验证人的选举，一段时间之后，选举得出的新验证人将会开始生产和验证区块，在这一阶段：

* 可以根据网络情况，调节验证人数量，既要保证一定程度的去中心化，也要确保网络的时延、区块生产间隔稳定、可靠；
* 使用 sudo 权限取消针对验证人的不必要惩罚。

### 启用治理功能

通过修改`BaseCallFilter`的逻辑，可以启用所需的治理功能，包括：

* 投票选举议会成员；
* 议会成员通过提案维护网络；
* 网络用户通过提案申请国库资金，不过需要等待转账功能开启才可以真正转账；
* 通过公投提案升级网络等等。

### 删除 sudo 权限

在网络的早期，合理地使用 sudo 权限，可以高效地管理网络的运转；但是在一个去中心化的网络中，这样的一个超级管理员，不符合去中心的价值观，在合适的时间点要进行删除，通常是在开启治理的一段时间之后。

删除 sudo 需要使用链上升级的方式，也就要求对此升级进行全民公投，由公投结果来决定是否删除 sudo 模块。

### 开启转账和其它功能模块

接着就可以一步步地引入和开启更多的功能模块，如转账、链上身份注册等等。

## 总结

通过本文，你已经了解到了如何修改 node-template 发布一个可用的公开测试网节点程序，chain spec 的组成和配置方法，最后我们模拟了如何渐进地启动一个公开测试网络。注意，**本文所列代码未经过审计，不可直接应用于生产环境。**
