
---
date: 2021-02-08
title: Substrate 如何使用 libp2p 进行点对点通信
tags: ["Blockchain", "Substrate"]
categories: ["Blockchain"]
---

通过本文，你将了解到，

* 点对点通信的特点；
* libp2p的基本介绍；
* 以及Substrate如何使用libp2p实现的点对点通信。

## 点对点通信

在Web2.0时代，绝大多数互联网应用采用了基于[TCP/IP](https://en.wikipedia.org/wiki/Internet_protocol_suite)的“客户端-服务器”的通信架构，在客户端采集数据并发送给服务器，服务器存储和处理数据，客户端进而获取并使用这些处理后的数据。这种模式支撑了互联网近三十年的蓬勃发展，在给普通用户提供便利的同时，也出现了各种各样的问题，比如：

* 泄露用户隐私；
* 贩卖用户数据；
* 服务商发布没有底线的广告给不明真相的用户造成不可挽回的损失；
* 不经协商，随意删除用户发布的内容；
* 垄断市场和定价；
* 过度利用用户心理，无节制地占据用户的时间；
* ......

![client-server](/static/libp2p/client_server.png)

在为以上问题寻找解决方案时，点对点（即peer to peer）的通信机制逐渐走进了技术先驱们的视野。在互联网早期的时候，点对点通信主要用于文件共享，如音乐共享服务Napster和流媒体下载服务BitTorrent。点对点的服务更加广泛的使用，还需要一定的治理机制，来处理资源的版权问题和现实世界的监管，此类内容不是本文的重点，不做过多地介绍。

![peer-to-peer](/static/libp2p/peer_to_peer.png)

在点对点的网络里，所有的节点都是对等的，即任何节点都可以存储和处理数据（作为服务端）；也可以发送待处理的数据给网络中的其它节点，获取经过网络处理后的数据（作为客户端）。通过这样的通信机制，可以保证，

* 网络具备开放性，节点可以自由加入和退出；
* 不依赖单一服务节点，网络的服务更加可靠、高效；
* 节点运行的程序代码公开可见，规则更加透明。

根据网络中传输的数据和提供服务的不同，点对点应用出现了不同的应用场景，包括文件存储和读取、数据计算、内容共享、数据交易等服务。在开发这些应用的过程中，可能涉及到的技术要点有：

* 节点身份，唯一地标识网络中的节点及地址格式；
* 发现机制，在没有中心化的协调服务存在的情况下，如何发现新的节点；
* 路由，本地节点无法存储网络中所有节点的信息，通过路由算法查找需要的节点；
* 多种通信协议比如TCP、UDP、WebSocket、QUIC等等；
* 加密和认证，保证消息的可靠和安全；
* NAT穿透，解决NAT后面的内部IP无法访问的难题；
* 多路复用以节省资源；
* 消息订阅，高效的获取更新而不会给网络造成负担；
* 中继，当需要建立通信的两个节点都无法直接被访问，比如都在NAT网络中，需要通过中继节点传递信息；

* ......

以上列出的这些技术要点/需求并不会出现在每个点对点应用里，大多数只会使用其中的一部分功能，尽管如此，还是存在严重地重复造轮子的现象。也有一些应用为了避免重复开发，选择了fork已有开源应用的功能代码，这种方式引入了原有应用的技术债，难于定制和扩展。

复杂多变的网络拓扑和膨胀的应用状态导致了点对点应用的开发、推广和普及都极为困难，出现一个高度模块化的点对点通信开发框架也就不足为奇，也就是接下来我们要了解的libp2p。

## Libp2p 介绍

Libp2p是一个开发点对点应用的框架，它最早源于去中心的文件共享服务IPFS，把网络通信相关的内容抽离并重新设计，形成了现在的libp2p，目前比较成熟的几个语言版本包括js-libp2p、go-libp2p、rust-libp2p，并且定义了一套参考规范，不同语言的实现版本只要符合这一参考规范，就可以实现互通信。

Libp2p提供的核心功能包括，

* 在节点之间建立**安全可复用**的网络连接；
* 可验证的节点身份和可连接的地址。

### 安全可复用的连接

Libp2p支持的底层（传输层）协议包括TCP/IP、UDP、WebSocket、QUIC等，不同语言版本的实现完成度不尽相同。连接的安全性是通过对传输内容进行加密来保证的，节点的身份也会进行相应的验证。

为了提升连接的利用率以及应对复杂的网络场景如各种形式的防火墙和NAT，对建立的底层连接进行多路复用十分有必要，stream就是可实现复用的一种上层连接形式，它可以是双向的，也可以是单向的。

QUIC协议有内置的安全和复用组件，对于没有此类功能的协议，使用libp2p可以对原始连接进行upgrade，添加所需的安全和可复用的套件，安全套件有[secio](https://docs.rs/libp2p-secio/0.22.0/libp2p_secio/)、[Noise](https://docs.rs/libp2p/0.28.1/libp2p/noise/index.html)，可复用套件有[yamux](https://docs.rs/libp2p-yamux/0.25.0/libp2p_yamux/)和[mplex](https://docs.rs/libp2p-mplex/0.22.0/libp2p_mplex/)。

![connection upgrade](/static/libp2p/connection_upgrade.svg)

在stream里可以传输各种各样的libp2p内置或用户自定义的[应用层协议](https://docs.libp2p.io/concepts/protocols/)，这些协议定义了节点间交换信息的方式和内容，比如:

* [ping](https://docs.libp2p.io/concepts/protocols/#ping)，用来定时检查节点是否在线；
* [identity](https://docs.libp2p.io/concepts/protocols/#identify)，用于节点间交换信息如节点的public key和网络中的地址；
* [kad-dht](https://docs.libp2p.io/concepts/protocols/#kad-dht)，基于Kademlia算法的分布哈希表，用于节点间路由；
* ......

以identity协议为例，它的协议id（具有路径格式的字符串）为`/ipfs/id/1.0.0`，消息的表示和序列化使用的是[protocol buffer](https://developers.google.com/protocol-buffers/)，

```protobuf
message Identify {
  optional string protocolVersion = 5;
  optional string agentVersion = 6;
  optional bytes publicKey = 1;
  repeated bytes listenAddrs = 2;
  optional bytes observedAddr = 4;
  repeated string protocols = 3;
}
```



### 节点身份

节点启动时需要提供一个private key（也可以随机生成），主要用于

* 将节点双方的公钥通过[Diffie-Helman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)对消息进行加解密；
* 对节点的public key进行哈希，生成PeerId即节点身份。

Libp2p支持的[公钥加密算法](https://github.com/libp2p/specs/blob/master/peer-ids/peer-ids.md#key-types)包括RSA、Ed25519、Secp256k1等。PeerId的生成采用了[multihashes](https://docs.libp2p.io/reference/glossary/#multihash)的形式，即支持多种哈希算法，经过base 58 编码后的格式如`QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N`。

将PeerId与[multiaddr](https://docs.libp2p.io/reference/glossary/#multiaddr)结合可以用来在网络中定位节点和验证身份，例如IP地址为7.7.7.7、监听在4242端口、拥有上述PeerId的节点的multiaddr（多层次地址）为：

```
ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N
```

以上只列出了libp2p提供的部分功能，更多内容例如消息订阅、中继、NAT穿透等等可以参考相关[文档](https://docs.libp2p.io/concepts/)，使用libp2p开发点对点应用可以解决以上提到的大部分难题/技术点，节约大量的开发时间，增加系统的可维护性和可扩展性。使用libp2p的应用包括IPFS，Substrate/Polkadot，Libra，Ethereum 2.0等等，接下来我们来了解下Substrate如何使用的libp2p。

### 简单应用

这里我们基于rust-libp2p，编写一个简单的点对点应用，可以完成回声（echo）的功能，即其中一个节点发送一个字符串，另一个节点接收该字符串并回复相同的字符串，这里我们需要自定义一个应用层的协议`EchoProtocol`，需要实现libp2p提供的`UpgradeInfo`接口，

```rust
#[derive(Default, Debug, Copy, Clone)]
pub struct EchoProtocol;

impl UpgradeInfo for EchoProtocol {
    type Info = &'static [u8];
    type InfoIter = iter::Once<Self::Info>;

    fn protocol_info(&self) -> Self::InfoIter {
        iter::once(b"/ipfs/echo/1.0.0")
    }
}
```

这里的`protocol_info`方法返回了协议的名字和格式。接着实现`InboundUpgrade`和`OutboundUpgrade`，这两个接口都继承自`UpgradeInfo`，

```rust
impl InboundUpgrade<NegotiatedSubstream> for EchoProtocol {
    type Output = NegotiatedSubstream;
    type Error = Void;
    type Future = future::Ready<Result<Self::Output, Self::Error>>;

    fn upgrade_inbound(self, stream: NegotiatedSubstream, _: Self::Info) -> Self::Future {
        future::ok(stream)
    }
}

impl OutboundUpgrade<NegotiatedSubstream> for EchoProtocol {
    type Output = NegotiatedSubstream;
    type Error = Void;
    type Future = future::Ready<Result<Self::Output, Self::Error>>;

    fn upgrade_outbound(self, stream: NegotiatedSubstream, _: Self::Info) -> Self::Future {
        future::ok(stream)
    }
}
```

`NegotiatedSubstream`表示协商好的某个协议将会使用的I/O流。当远端的节点支持当前协议时，调用`upgrade_inbound`和`upgrade_outbound`分别在listener和dialer端开启握手信号。

之后，定义处理连接请求的handler，也就是我们这里的结构体`EchoHandler`，它保存了处理过程中所使用的状态信息。

```rust
pub struct EchoHandler {
    inbound: Option<EchoFuture>,
    outbound: Option<EchoFuture>,
    init_echo: bool,
    already_echo: bool,
}

type EchoFuture = BoxFuture<'static, Result<NegotiatedSubstream, io::Error>>;

impl EchoHandler {
    pub fn new(init_echo: bool) -> Self {
        EchoHandler {
            inbound: None,
            outbound: None,
            init_echo,
            already_echo: false,
        }
    }
}
```

还需要一个自定义的枚举event枚举类型，

```rust
#[derive(Debug)]
pub enum EchoHandlerEvent {
    Success,
}
```

接着就可以实现libp2p::swarm里所提供的`ProtocolsHandler`接口了，

```rust
impl ProtocolsHandler for EchoHandler {
    type InEvent = Void;
    type OutEvent = EchoHandlerEvent;
    type Error = ReadOneError;
    type InboundProtocol = EchoProtocol;
    type OutboundProtocol = EchoProtocol;
    type OutboundOpenInfo = ();
    type InboundOpenInfo = ();

    fn listen_protocol(&self) -> SubstreamProtocol<EchoProtocol, ()> {
        SubstreamProtocol::new(EchoProtocol, ())
    }

    fn inject_fully_negotiated_inbound(&mut self, stream: NegotiatedSubstream, (): ()) {
        if self.inbound.is_some() {
            panic!("already have inbound");
        }
        log::debug!("ProtocolsHandler::inject_fully_negotiated_inbound");
        self.inbound = Some(recv_echo(stream).boxed());
    }

    fn inject_fully_negotiated_outbound(&mut self, stream: NegotiatedSubstream, (): ()) {
        if self.outbound.is_some() {
            panic!("already have outbound");
        }
        log::debug!("ProtocolsHandler::inject_fully_negotiated_outbound");
        self.outbound = Some(send_echo(stream).boxed());
    }

    fn inject_event(&mut self, _: Void) {
    }

    fn inject_dial_upgrade_error(&mut self, _info: (), error: ProtocolsHandlerUpgrErr<Void>) {
        log::debug!("ProtocolsHandler::inject_dial_upgrade_error: {:?}", error);
    }

    fn connection_keep_alive(&self) -> KeepAlive {
        KeepAlive::Yes
    }

    fn poll(&mut self, cx: &mut Context<'_>) -> Poll<
        ProtocolsHandlerEvent<
            EchoProtocol,
            (),
            EchoHandlerEvent,
            Self::Error
        >
    > {
        if let Some(fut) = self.inbound.as_mut() {
            match fut.poll_unpin(cx) {
                Poll::Pending => {
                    log::debug!("ProtocolsHandler::poll, inbound is some but pending...");
                }
                Poll::Ready(Err(e)) => {
                    log::error!("ProtocolsHandler::poll, inbound is some but resolve with error: {:?}", e);
                    self.inbound = None;
                    panic!();
                }
                Poll::Ready(Ok(stream)) => {
                    self.inbound = Some(recv_echo(stream).boxed());
                    return Poll::Ready(ProtocolsHandlerEvent::Custom(EchoHandlerEvent::Success))
                }
            }
        }

        match self.outbound.take() {
            Some(mut send_echo_future) => {
                match send_echo_future.poll_unpin(cx) {
                    Poll::Pending => {
                        // The future has not yet finished. Make sure
                        // to poll it again on the next iteration.
                        self.outbound = Some(send_echo_future);
                    },
                    Poll::Ready(Ok(_stream)) => {
                        return Poll::Ready(
                            ProtocolsHandlerEvent::Custom(
                                EchoHandlerEvent::Success
                            )
                        )
                    },
                    Poll::Ready(Err(e)) => {
                        log::error!("ProtocolsHandler::poll, outbound is some but resolve with error: {:?}", e);
                        panic!();
                    }
                }
            },
            None => {
                if self.init_echo && !self.already_echo {
                    self.already_echo = true;
                    let protocol = SubstreamProtocol::new(EchoProtocol, ());
                    return Poll::Ready(ProtocolsHandlerEvent::OutboundSubstreamRequest {
                        protocol
                    })
                }
            },
        }
        
        Poll::Pending
    }

}
```

**当节点为dialer**，handler在轮询（`ProtocolsHandler::poll()`）时，需要返回包含`EchoProtocol`实例的`ProtocolsHandlerEvent::OutboundSubstreamRequest`，用于发起并协商连接使用的协议。如果协商成功，调用`ProtocolsHandler::inject_fully_negotiated_outbound`，在这里我们将handler保存的outbount状态由None更新为`Some(send_echo(stream).boxed())`，其中`send_echo`接收协商好的IO stream，无错误发生时返回该stream。

```rust
const ECHO_SIZE: usize = 12;

pub async fn send_echo<S>(mut stream: S) -> io::Result<S>
where
    S: AsyncRead + AsyncWrite + Unpin
{
    // mxinden: A bit of a hack. Likely nicer to do somewhere else.
    futures_timer::Delay::new(std::time::Duration::from_secs(3)).await;

    let payload = "hello world!";
    log::info!("send_echo, preparing send payload: {:?}, in bytes: {:?}", payload, payload.as_bytes());
    stream.write_all(payload.as_bytes()).await?;
    stream.flush().await?;
    let mut recv_payload = [0u8; ECHO_SIZE];
    log::info!("send_echo, awaiting echo for {:?}", payload);
    
    stream.read_exact(&mut recv_payload).await?;
    log::info!("send_echo, received echo: {:?}", str::from_utf8(&recv_payload));
    if str::from_utf8(&recv_payload) == Ok(payload) {
        Ok(stream)
    } else {
        Err(io::Error::new(io::ErrorKind::InvalidData, "Echo payload mismatch"))
    }
}
```

我们还接着看`ProtocolsHandler::poll`里的实现，当outbound为Some，send_echo返回的future轮询的结果为`Poll::Pending`时，更行outbound为`self.outbound = Some(send_echo_future)`，保证下次轮询时依然有效，当结果为`Poll::Ready`时返回相应的事件信息。

**当节点为listener**，连接中出现新的请求流时，自动调用`ProtocolsHandler::listen_protocol`返回一个`InboundUpgrade`的实例来协商流使用的协议。协商成功之后，调用`inject_fully_negotiated_inbound`，其中一个参数为协商好的stream，在该方法内，将handler的inbound属性状态更新为`Some(recv_echo(stream).boxed())`，`recv_echo`方法的实现为，

```rust
pub async fn recv_echo<S>(mut stream: S) -> io::Result<S>
where
    S: AsyncRead + AsyncWrite + Unpin
{
    let mut payload = [0u8; ECHO_SIZE];
    log::info!("recv_echo, waiting for echo...");
    stream.read_exact(&mut payload).await?;
    log::info!("recv_echo, receive echo request for payload: {:?}", payload);
    stream.write_all(&payload).await?;
    stream.flush().await?;
    log::info!("recv_echo, echo back successfully for payload: {:?}", payload);
    Ok(stream)
}
```

这里泛型`S`需要满足[futures_io](https://docs.rs/futures-io/0.3.12/futures_io/)提供的`AsyncRead`和`AsyncWrite`约束。

点对点网络就像一个蜂群（Swarm），而蜂群的整体行为是由单一个体的行为所组成的，单一个体的行为由一系列的规则所制定，此类的规则可以组合使用，在rust-libp2p中，规定的定义需要实现`NetworkBehaviour`接口，这里我们先定义一个结构体，对规则的状态进行保存。

```rust
pub struct EchoBehaviour {
    events: VecDeque<EchoBehaviourEvent>,
    config: EchoBehaviourConfig,
}

pub struct EchoBehaviourConfig {
    init_echo: bool,
}

impl EchoBehaviour {
    pub fn new(config: EchoBehaviourConfig) -> Self {
        EchoBehaviour {
            events: VecDeque::new(),
            config,
        }
    }
}

#[derive(Debug)]
pub struct EchoBehaviourEvent {
    pub peer: PeerId,
    pub result: EchoHandlerEvent,
}
```

本结构体包含了与Swarm沟通的消息`events`，行为定义所需要的初始配置。接着，就可以实现`NetworkBehaviour`接口了，

```rust
impl NetworkBehaviour for EchoBehaviour {
    type ProtocolsHandler = EchoHandler;
    type OutEvent = EchoBehaviourEvent;

    fn new_handler(&mut self) -> Self::ProtocolsHandler {
        EchoHandler::new(self.config.init_echo)
    }

    fn addresses_of_peer(&mut self, _peer_id: &PeerId) -> Vec<Multiaddr> {
        Vec::new()
    }

    fn inject_connected(&mut self, _: &PeerId) {
        log::debug!("NetworkBehaviour::inject_connected");
    }

    fn inject_disconnected(&mut self, _: &PeerId) {
        log::debug!("NetworkBehaviour::inject_disconnected");
    }

    fn inject_event(&mut self, peer: PeerId, _: ConnectionId, result: EchoHandlerEvent) {
        log::debug!("NetworkBehaviour::inject_event");
        self.events.push_front(EchoBehaviourEvent { peer, result })
    }

    fn poll(&mut self, _: &mut Context<'_>, _: &mut impl PollParameters)
        -> Poll<NetworkBehaviourAction<Void, EchoBehaviourEvent>>
    {
        log::debug!("NetworkBehaviour::poll, events: {:?}", self.events);
        if let Some(e) = self.events.pop_back() {
            Poll::Ready(NetworkBehaviourAction::GenerateEvent(e))
        } else {
            Poll::Pending
        }
    }
}
```

当连接建立或者尝试去呼叫节点时会调用`new_handler`，返回我们之前定义的handler即`EchoHandler`，作为该连接的后台处理线程，behaviour和handler通过消息传递的机制进行通信，`inject_event`可以把handler的消息传给behaviour，behaviour在poll的时候返回`SendEvent`将消息传递给handler。

到这里，我们已经完成了一个简单的echo点对点通信的协议，现在我们看一下main函数里如何使用。

```rust
fn main() -> Result<(), Box<dyn Error>> {
    env_logger::init();

    // create a random peerid.
    let id_keys = identity::Keypair::generate_ed25519();
    let peer_id = PeerId::from(id_keys.public());
    log::info!("Local peer id: {:?}", peer_id);

    // create a transport.
    let transport = libp2p::build_development_transport(id_keys)?;
    
    let mut behaviour_config = EchoBehaviourConfig { init_echo: false };
    // get the multi-address of remote peer given as the second cli argument.
    let target = std::env::args().nth(1);
    // if remote peer exists, the peer can initialize an echo request.
    if target.is_some() {
        behaviour_config = EchoBehaviourConfig { init_echo: true };
    }

    // create a echo network behaviour.
    let behaviour = EchoBehaviour::new(behaviour_config);
    
    // create a swarm that establishes connections through the given transport
    // and applies the echo behaviour on each connection.
    let mut swarm = Swarm::new(transport, behaviour, peer_id);

    // if the remote peer exists, dial it.
    if let Some(addr) = target {
        let remote = addr.parse()?;

        Swarm::dial_addr(&mut swarm, remote)?;
        log::info!("Dialed {}", addr)
    }
    

    // Tell the swarm to listen on all interfaces and a random, OS-assigned port.
    Swarm::listen_on(&mut swarm, "/ip4/0.0.0.0/tcp/0".parse()?)?;

    let mut listening = false;
    task::block_on(future::poll_fn(move |cx: &mut Context<'_>| {
        loop {
            match swarm.poll_next_unpin(cx) {
                Poll::Ready(Some(event)) => log::info!("Get event: {:?}", event),
                Poll::Ready(None) => {
                    log::info!("Swam poll next ready none");
                    return Poll::Ready(())
                },
                Poll::Pending => {
                    if !listening {
                        for addr in Swarm::listeners(&swarm) {
                            log::info!("Listening on {}", addr);
                            listening = true;
                        }
                    }
                    return Poll::Pending
                }
            }
        }
    }));

    Ok(())
}
```

代码的简单说明如下：

* 通过`Keypair::generate_ed25519`生成用于节点间通信加密的密钥，其中的公钥可以派生出节点的`PeerId`。
* `libp2p::build_development_transport`构建了开发常用的传输层，支持TCP/IP、WebSocket，使用noise协议作为加密层，yamux和mplex多路复用协议。
* 解析传入参与，如果包含呼叫的节点信息，则是dialer（客户端），将构造behaviour的初始参数`init_echo`设置为true。
* 使用上面构造的传输层、behaviour、节点id，调用`Swarm::new(transport, behaviour, peer_id)`构造模拟网络的swarm。
* 当节点为dialer时，呼叫传入的远端节点`Swarm::dial_addr(&mut swarm, remote)?`，将该节点加入swarm节点池中。
* 对swarm进行轮询`swarm.poll_next_unpin(cx)`，如果有behaviour触发消息，处理对应的消息。

**小结**，libp2p对点对点通信进行了高度的抽象，在开始接触这些概念时，容易摸不着头脑，需要不断去熟悉划分的层次和常用的协议；rust-libp2p的实现，针对libp2p定义的层次和协议，封装出了不同的接口，在开发自定义协议的同时，需要深入去了解这些抽象的接口及接口间通信的方式。总体来说，点对点通信开发的难度比传统的客户端-服务器通信形式高很多，libp2p的设计在于弥合这其中的一些痛点，但也还有很长的路要走，应用开发者也需要更多的了解底层的机制才能更好的开发应用协议。



## Substrate 网络层

区块链网络是由去中心（或者说是点对点）的节点所组成，节点之间通过网络连接传递消息，Substrate作为通用的区块链开发框架，它的网络层使用了rust-libp2p，可以很容易地使用、扩展一系列的通信协议，例如：

* 传输层支持TCP/IP（地址格式为`/ip4/1.2.3.4/tcp/5`）、WebSocket（地址格式为`/ip4/1.2.3.4/tcp/5/ws`）、DNS（地址格式`/dns/example.com/tcp/5`或`/dns/example.com/tcp/5/ws`），以及对应的IPv6格式；
* 在传输层之上应用了加密协议 [Noise](https://docs.rs/libp2p/0.34.0/libp2p/noise/index.html)；
* 支持多路复用协议[Yamux](https://github.com/hashicorp/yamux/blob/master/spec.md)和[Mplex](https://github.com/libp2p/specs/tree/master/mplex)，其中mplex会逐步废弃；
* 使用libp2p标准的[ping协议](https://docs.rs/libp2p/0.34.0/libp2p/ping/index.html)（`/ipfs/ping/1.0.0`），周期性的检查节点间的网络连接是否还活着，如果检查失败会断开连接；
* 使用libp2p标准的[id协议](https://docs.rs/libp2p/0.34.0/libp2p/identify/index.html)（`/ipfs/id/1.0.0`），节点之间通过该协议周期性地交换节点各自的信息；
* libp2p标准的[Kademlia协议](https://docs.rs/libp2p/0.34.0/libp2p/kad/index.html)（`/<protocol_id>/kad`），执行Kademlia random walk查询，其中protocol_id可以用来区分不同的链，在Substrate chain spec中进行定义；
* 自定义的[sync协议](https://crates.parity.io/sc_network/index.html#substreams)（`/<protocol-id>/sync/2`），用来同步区块信息，请求和返回结果的数据格式定义在[api.v1.proto](https://github.com/paritytech/substrate/blob/master/client/network/src/schema/api.v1.proto)文件中；
* 自定义的[light协议](https://crates.parity.io/sc_network/index.html#substreams)（`/<protocol-id>/light/2`），轻客户端用此协议同步链上的状态信息，数据格式定义在[light.v1.proto](https://github.com/paritytech/substrate/blob/master/client/network/src/schema/light.v1.proto)文件中；
* 自定义的[transactions协议](https://crates.parity.io/sc_network/index.html#substreams)（`/<protocol-id>/transactions/1`），用来广播节点接收到的交易信息，它的格式是交易集合的SCALE编码结果；
* 自定义的[区块广播协议](https://crates.parity.io/sc_network/index.html#substreams)（`/<protocol-id>/block-announces/1`），当节点产出或者接收到区块时，将此区块向其它节点进行广播；
* 自定义的[gossip协议](https://crates.parity.io/sc_network/index.html#substreams)（`/paritytech/grandpa/1`），GRANDPA用来通知其它节点相关的投票信息；
* 自定义的[Substrate legacy协议](https://crates.parity.io/sc_network/index.html#the-legacy-substrate-substream)（`/substrate/<protocol-id>/<version>`），是一个即将被弃用的协议，它也可以同步、广播区块信息，处理轻客户端请求，Gossiping（被GRANDPA使用）等等。

结合以上的底层和应用层通信协议，Substrate的节点之间可以通过三种发现机制方式建立起连接，

* 启动节点（bootstrap nodes），它的地址和PeerId都是固定的，适用于网络的冷启动和某节点刚加入网络时，通过启动节点进入到网络中；
* [mDNS](https://github.com/libp2p/specs/blob/master/discovery/mdns.md)，在本地网络通过广播UDP数据包，如果有节点响应，则可以建立起连接；
* Kademlia random walk，当连接建立后，当前节点可以通过`FIND_NODE` 请求远端节点，获取远端节点对当前网络中节点的视角。

以上的协议共同构成了Substrate的通用网络层，而这一网络层的使用是通过`NetworkWorker`和`NetworkService`结构体来实现的，在node template节点程序中的使用示例如下：

```rust
let (network, network_status_sinks, system_rpc_tx, network_starter) =
		sc_service::build_network(sc_service::BuildNetworkParams {
			config: &config,
			client: client.clone(),
			transaction_pool: transaction_pool.clone(),
			spawn_handle: task_manager.spawn_handle(),
			import_queue,
			on_demand: None,
			block_announce_validator_builder: None,
		})?;
```



## 总结

去中心的通信模型为互联网应用开辟了新的范式，也带来了相当大的挑战，libp2p规范的出现，逐步减轻了开发者在开发点对点应用所遇到的痛点。Substrate借助libp2p的优良特性，在区块链这一细分的去中心应用领域，可以很方便的让普通开发者无需过多的关注底层的通信机制，就可以完成复杂的自定义区块链应用。



## Reference

[Wiki: Peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer)

[How to Leverage Libp2p for Blockchain Applications](https://www.infoq.com/presentations/blockchain-libp2p/)

[Substrate Crate sc_network](https://crates.parity.io/sc_network/index.html)

[IPFS secure channel](https://github.com/auditdrivencrypto/secure-channel/blob/master/prior-art.md#ipfss-secure-channel)

[Libp2p specification](https://github.com/libp2p/specs)

[Textile's primer on libp2p](https://blog.textile.io/a-primer-on-libp2p/)

