---
date: 2020-04-02
title: Substrate代码导读：node-template
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

通过本文，你会了解到，

* Substrate node-template的组成部分，及各部分的功能简介
* 参数如何配置

Substrate作为一个标准的区块链开发框架，不仅提供了必备的底层公共组件（如数据库、共识、P2P、交易池）和通用的runtime模块（如资产相关的balances，治理相关的democracy等），还提供了将各个功能组件连接起来的节点模板程序（[node-template](https://github.com/paritytech/substrate/tree/master/bin/node-template)）和节点程序（[node](https://github.com/paritytech/substrate/tree/master/bin/node)）。本文主要介绍node-template中各个代码块的功能。

## 文件目录

在使用类Unix操作系统的情况下，进入node-template文件目录，执行`tree -I target`命令，获取详细的文件信息如下：

![node_template_tree_info](/static/node-template/node_template_dir.png)

> 这里我们忽略了target文件目录下的内容，来较少干扰性的输出。

## workspace cargo.toml

node-template是一个标准的[Rust workspace](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)项目，当项目比较复杂时，使用workspace可以清晰地管理组件库（library）和可执行程序（binary）。在项目根目录的cargo.toml文件里有：

```toml
[workspace]
members = [
    'node',
    'pallets/template',
    'runtime',
]
```

这个workspace的成员有node、pallets/template、runtime，其中node是可执行程序，在对应的src/main.rs文件内拥有一个可执行的main函数入口；pallets/template和runtime是组件库，在src/lib.rs定义了可被外部使用的函数和数据结构。

细心的同学会注意到cargo.toml里还有下面两行配置:

```toml
[profile.release]
panic = 'unwind'
```

它和`catch_unwind`一起使用可以捕获某个线程内panic抛出的异常，常用的场景有：

* 在其它编程语言中嵌入Rust；
* 自定义线程处理的逻辑；
* 测试框架，因为测试用例可以panic，但是不能中断测试的运行。

具体请参考[Controlling panics with `std::panic`](https://doc.rust-lang.org/edition-guide/rust-2018/error-handling-and-panics/controlling-panics-with-std-panic.html)。

## workspace build.rs

自定义的构建脚本放置在项目的build.rs文件内，可以在编译构建项目之前，让Cargo去编译和执行该脚本，使用场景有：

* 编译、连接第三方的非Rust代码；
* 构建之前的代码生成功能。

node-template根目录下的build.rs的具体功能，参考下面的注释，

```rust
use vergen::{ConstantsFlags, generate_cargo_keys};

const ERROR_MSG: &str = "Failed to generate metadata files";

fn main() {
	// 使用vergen生成环境变量，供项目中的env!宏获取
  // 这里设置了VERGEN_SHA_SHORT为Git最新的commit id的缩写
	generate_cargo_keys(ConstantsFlags::SHA_SHORT).expect(ERROR_MSG);

  // 当.git/HEAD文件改变即切换Git分支时，重新执行这一脚本
	build_script_utils::rerun_if_git_head_changed();
}
```

## scripts/init.sh

初始化编译环境，包括升级Rust的版本，包括nightly和stable两个[发布渠道](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#choo-choo-release-channels-and-riding-the-trains)：

```shell
rustup update nightly
rustup update stable
```

并且添加构建WebAssembly的支持工具：

```shell
rustup target add wasm32-unknown-unknown --toolchain nightly
```

定期执行本脚本，可以解决一些常见的编译问题如某个依赖安装失败。

## pallets

包含了自定义的runtime模块，默认只有一个template模块，以此模块为例：

**[cargo.toml](https://doc.rust-lang.org/cargo/reference/manifest.html)包含**：

* package的基本信息如name，version，authors等。

* package所依赖的第三方库，以`frame-support`为例，来源是Github上该代码仓库的某个commit id，并将default-features设置为false（即不使用默认的feature进行编译）。

  ```toml
  [dependencies.frame-support]
  default-features = false
  git = 'https://github.com/paritytech/substrate.git'
  rev = '013c1ee167354a08283fb69915fda56a62fee943'
  version = '2.0.0-alpha.3'
  ```

* 通过[feature](https://doc.rust-lang.org/cargo/reference/features.html)进行条件编译，当使用Cargo进行构建时，下面的配置表示默认使用std feature，当编译依赖库如frame-support也默认使用std feature。这样的配置保证了runtime模块既可以编译为Native执行版本（使用std feature），也可以编译为Wasm执行版本（使用no_std feature，并由[WasmBuilder](https://github.com/paritytech/substrate/tree/master/utils/wasm-builder)进行编译，后面会详细介绍）。

  ```toml
  [features]
  default = ['std']
  std = [
      'codec/std',
      'frame-support/std',
      'safe-mix/std',
      'system/std',
  ]
  ```

>说明：Substrate为了保证应用的安全和稳定，对runtime有意地添加了一个约束，也就是在runtime代码里只能使用Rust的核心库及一些辅助库，而不能使用[标准库](https://doc.rust-lang.org/std/)。使用标准库会导致Wasm执行版本编译失败。

**src/lib.rs** 是runtime模块的具体功能实现：

* `#![cfg_attr(not(feature = "std"), no_std)]`表示编译时如果feature不是std，那么必须是no_std。

* mock和test模块只在运行测试时进行编译；

* 定义了模块的接口，继承自system模块的接口，并添加了一个关联类型Event，这个Event类型可以转换成system模块下的Event，也可以由当前的template模块定义的的Event转换而来；

  ```rust
  pub trait Trait: system::Trait {
      type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
  }
  ```

* 使用decl_storage宏定义模块的存储单元；
* 使用decl_event宏定义模块可能触发的事件；
* 使用decl_error宏定义模块可以返回的错误种类；
* 最后通过decl_module宏定义了本模块的核心逻辑即可调用函数（Dispatchable Call），并初始化Error类型和Event的默认触发方式。

**src/mock.rs** 是为测试用例服务的，

* `pub struct Test;`创建了一个测试用的runtime结构体；

* 通过`impl_outer_origin`宏为runtime构造了一个Origin类型，用来标识交易的来源；

* 通过`parameter_types`宏生成一些后面功能模块所需的满足`Get`接口的数据类型；

* 为runtime实现各个功能模块接口，这里使用了大量的`()`来mock不关心的数据类型；

  ```rust
  impl Trait for Test {
      type Event = ();
  }
  ```

* `pub type TemplateModule = Module<Test>;`定义别名，使测试代码更简洁；

* 定义了函数`new_test_ext`来初始化GenesisConfig，并返回一个基于HashMap的TestExternalities，用于存储的读写和其它扩展操作。

**src/tests.rs** 包含了所有的测试用例，

* 引入mock数据和断言；
* 通过`#[test]`来标识测试函数；
* 调用`new_test_ext`，并通过`execute_with`执行closure内的代码；
* 调用template模块的可调用函数，并返回执行结果`TemplateModule::do_something(Origin::signed(1)`。
* `assert_ok`断言结果是Ok，`assert_eq`断言结果等于预期，`assert_noop`断言结果为Error并且不修改链上存储状态。



## runtime

**cargo.toml** 除了上面提到的某个pallet对应的cargo.toml里的类似内容外，还添加了，

* 构建脚本即build.rs的依赖`wasm-builder-runner`；

* Substrate内置和开发者自定义的runtime模块（也叫做pallet）。

  ```toml
  [dependencies.sudo]
  default-features = false
  git = 'https://github.com/paritytech/substrate.git'
  package = 'pallet-sudo'
  rev = '013c1ee167354a08283fb69915fda56a62fee943'
  version = '2.0.0-alpha.3'
  
  [dependencies.template]
  default-features = false
  package = 'pallet-template'
  path = '../pallets/template'
  version = '2.0.0-alpha.3'
  ```

**build.rs** 使用`wasm-builder-runner`将当前的runtime项目编译为Wasm，编译后的文件位于`target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm`。

**src/lib.rs** 就是构造我们链上runtime的入口，

* `#![cfg_attr(not(feature = "std"), no_std)]`表示编译时如果feature不是std，那么必须是no_std；

* `#![recursion_limit="256"]`用来设置编译期间可能出现无限递归操作（如宏展开）的最大阈值，默认是128，这里我们提高到256，来满足`consturct_runtime`宏的需求；

* 使用std featue编译时，将生成的Wasm二进制内容通过常量的方式引入到当前runtime代码中。

  ```rust
  #[cfg(feature = "std")]
  include!(concat!(env!("OUT_DIR"), "/wasm_binary.rs"));
  ```

* 引入依赖的模块，以及为了下游模块方便调用而暴露上游模块的部分函数和数据类型，如`pub use balances::Call as BalancesCall`；

* 引入template模块 `pub use template`；

* 给runtime所需的基础类型起别名，原则是和模块中的associate type名称一致，如`pub type BlockNumber = u32`；

* opaque模块封装了一些用于CLI初始化时的类型，这些类型和runtime的具体信息；

* 指定了runtime版本信息，当runtime协议修改之后，需要将`spec_version`加1；`impl_version`是协议的实现版本，用来表示节点运行的代码是不同的，仅当非共识相关的优化发生时才可能修改这个值；`RUNTIME_API_VERSIONS`包含已实现的runtime api的所有版本信息，由`impl_runtime_apis`宏生成，后面会进一步介绍。

  ```rust
  pub const VERSION: RuntimeVersion = RuntimeVersion {
      spec_name: create_runtime_str!("node-template"),
      impl_name: create_runtime_str!("node-template"),
      authoring_version: 1,
      spec_version: 1,
      impl_version: 1,
      apis: RUNTIME_API_VERSIONS,
  };
  ```

* 定义区块时间相关的常量，如`pub const MILLISECS_PER_BLOCK: u64 = 6000`，即每个区块是6秒，可以根据需要修改配置；

* 指定当前的NativeVersion，在执行交易时会把NativeVersion和链上的RuntimeVersion进行比较，如果不一致，通常情况下会使用Wasm执行交易。

* 使用`parameter_types`宏生成一些后面功能模块所需的满足`Get`接口的数据类型；

* 为runtime的实现各个功能模块的接口，runtime由`construct_runtime`宏生成，

  ```rust
  impl template::Trait for Runtime {
      type Event = Event;
  }
  ```

* `construct_runtime`宏根据名称（如`TemplateModule`）和所用的模块内的组件（如`template::{Module, Call, Storage, Event<T>}`）来构造runtime，从而使模块中的信息通过metadata暴露出来，并且使该模块在runtime中可用。构造时，是按照顺序加载初始存储的，所以当B模块依赖A模块时，应当将A模块放在B之前。

* 通过`impl_runtime_apis`宏实现runtime api定义的接口，这些接口需要通过`decl_runtime_apis`宏进行定义。

## node

**cargo.toml** 使用`[[bin]]`表示这个包是可执行的，通过`build-dependencies`引入编译时的依赖，在build.rs中使用，其它内容在前面的章节已经介绍过了。

**build.rs** 的内容和workspace根目录下的build.rs相同。

**src/main.rs** 是node-template编译成可执行程序的入口文件，

* `#![warn(missing_docs)]`在编译时，当模块缺少文档时会打印warnning；

* 引入了当前目录下的其它代码模块，如`mod chain_spec`；
* `#[macro_use]`会加载引入的模块下的所有宏；
* main函数是程序的入口，它返回一个[自定义的Result类型](https://github.com/paritytech/substrate/blob/master/client/cli/src/error.rs#L20)，在函数内首先构造了一个`VersionInfo`的结构体，用来保存可执行程序的版本信息，其中`VERGEN_SHA_SHORT`是在编译时由build.rs产生的，然后执行command模块提供的run函数。

**src/command.rs** 提供了main所需的run函数，

* 通过`from_args`解析命令行的执行参数，返回一个`Cli`结构体，具体参考下面src/cli.rs的内容；
* 创建一个默认的[Substrate服务配置](https://substrate.dev/rustdocs/v2.0.0-alpha.3/sc_service/config/struct.Configuration.html)，这些服务包含启动线程运行网络、客户端和交易池等；
* 如果返回的`Cli`实例里存在子命令，则执行子命令，执行子命令时，
  * 首先进行初始化，如设置panic的异常处理机制，日志等；
  * 通过`chain_spec::load_spec`获取chain的配置，来更新前面构造的Substrate服务配置；
  * 调用子命令的`run`函数来执行该命令，run函数依赖src/service.rs模块里的`new_full_start`宏来返回ServiceBuilder，包含了构建Substrate服务的多种组件。
* 如果返回的`Cli`实例里没有子命令，则执行当前命令，
  * 首先初始化，和子命令的初始化功能一样；
  * 更新Substrate服务配置，比子命令的更新操作更全面，配置的所有属性都会更新；
  * 调用`run`来启动节点，需要传入全节点客户端的服务实例和轻节点客户端的服务实例，根据服务配置中的节点角色进行选择，启动完成后，保持运行直到接收到退出信号`SIGINT`（即Ctrl+C）。

**src/cli.rs** 借助[StructOpt](https://github.com/TeXitoi/structopt)库将命令行参数解析为Cli结构体，包含：

* 可选的子命令，如`purge-chain`清空本地存储，`build-spec`创建一个spec.json的初始文件，`revert`回滚链上状态等；
* 命令行参数，如`--validator`开启验证人模式，`--light`以轻客户端方式运行，`--ws-port 9944`指定WebSocket监听的TCP端口，等等。编译node-template之后，可以通过`./target/release/node-template -h`获取所有可用的子命令和参数，及其帮助信息。

```rust
#[derive(Debug, StructOpt)]
pub struct Cli {
	#[structopt(subcommand)]
	pub subcommand: Option<Subcommand>,

	#[structopt(flatten)]
	pub run: RunCmd,
}
```

**chain_spec.rs** 构造了`ChainSpec`，它定义了链的可用配置，用来构造初始区块，

* node-template提供了两种模式，通过命令行参数`--dev`指定开发者网络（Development），只有Alice是验证人；`--local`指定本地测试网络（LocalTestnet），Alice和Bob是验证人；
* 调用`ChainSpec::from_genesis`创建硬编码的ChainSpec；
* 定义了`testnet_genesis`函数，传入验证人列表、root账户、存有余额的账户列表，构造出GenesisConfig。

**service.rs** 提供了构造Substrate服务的帮助方法，

* 使用`native_executor_instance`宏定义了一个结构体`Executor`，并且实现了`NativeExecutionDispatch`接口，即可以通过函数名称来调用该函数；
* `new_full_start`宏构建了一个ServiceBuilder，用来构造全节点服务，过程如下：
  * 调用`with_select_chain `设置链的生成策略，也就是当链出现分叉的时候，选择哪个链继续工作，这里使用最长链原则；
  * 调用`with_transaction_pool` 设置交易池类型，这里使用BasicPool；
  * 调用`with_import_queue` 设置了导入区块所需的队列，这里使用BasicQueue，可以顺序地导入block。
* 使用`new_full`构建一个全节点服务，
  * 使用`new_full_start`宏构建了一个ServiceBuilder；
  * 调用`with_finality_proof_provider`设置使用何种策略提供最终性验证；
  * 调用`build`构建真正的[Substrate Service](https://substrate.dev/rustdocs/v2.0.0-alpha.3/sc_service/struct.Service.html)；
  * 如果是验证人并且不是哨兵模式，调用service的`spawn_essential_task`函数，启动用于生成区块的后台任务，使用的是Aura算法；
  * 如果GRANDPA功能没有关闭，调用`spawn_essential_task`开启后台运行的投票任务。
* 使用`new_light`构建一个轻节点服务，
  * 调用`with_select_chain `设置跟随最长链；
  * 调用`with_transaction_pool`设置BasicPool交易池类型;
  * 调用`with_import_queue_and_fprb`设置区块导入时所用的队列，以及用来构建最终性验证请求的`FinalityProofRequestBuilder`；
  * 调用`with_finality_proof_provider`设置使用何种策略提供最终性验证；
  * 调用`build`构建真正的[Substrate Service](https://substrate.dev/rustdocs/v2.0.0-alpha.3/sc_service/struct.Service.html)。

## 总结

通过本文，我相信你已经对node-template项目的代码有了比较深入的了解，现在你可以试着调节runtime代码中的一些配置参数如出块时间，自定义一条区块链。

## 更多

Substrate官方文档：[https://substrate.dev/](https://substrate.dev/)

Parity介绍：[https://www.parity.io/](https://www.parity.io/)

Substrate源码：[https://github.com/paritytech/substrate](https://github.com/paritytech/substrate)
