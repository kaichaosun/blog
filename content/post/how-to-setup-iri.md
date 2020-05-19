---
date: 2017-12-17
title: 如何部署 IOTA 的 IRI headless 全节点
tags: ["Blockchain"]
categories: ["Blockchain"]
---

## 什么是 IOTA
IOTA 的设计初衷是面向未来的机器与机器、人与人、人与机器之间点对点互联互通的DLT（分布式账本技术）协议。  
通过利用DAG实现了无交易费、高TPS，将whisper protocol用于消息传递，有效提高消息的可达性和私密性。  
本文所关注的IRI (IOTA Reference implementation)是IOTA协议Java版本的实现，基于Rust的版本正在开发之中。

## 部署 IRI headless 全节点

### 服务器选择
目前国内的服务器提供商包括阿里云，华为，青云等，
国外的有AWS，GCP（谷歌云服务），digital ocean等，价格不等，配置大同小异，本文将会以阿里云为例。  
运行 IRI 全节点配置至少是:

* 1vCPU  
* 4G 内存  
* 1M 带宽    
* 硬盘40G目前是足够的，可以按需添加数据盘  
* ECS 实例规格应为独享  

操作系统选择适合自己的Linux发行版，建议使用每种发行版的最新版本，如Ubuntu 16.04。

本文所使用配置，如下图：
![iri_aliyun_config](/static/iri-setup/iri_aliyun_config.png)

### 服务器的基础环境准备

1.安装Git，在IRI搭建时并没有用到，之后tangle网络生成snapshot时需要：

```
sudo apt-get update

sudo apt-get install git
```

2.安装screen，管理服务器终端产生的session

```
sudo apt-get install screen
```

3.安装Java，
Oracle JDK，添加Oracle的包管理仓库并更新，  

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
```

安装JDK8，

```
sudo apt-get install oracle-java8-installer
```
查看安装，

```
java -version
```

### 下载并运行IRI

1.IRI的Github release链接：https://github.com/iotaledger/iri/releases  
下载最新Mainnet版本到本地，截止目前最新版为：v1.4.1.2。

2.添加配置内容，保存至本地iri.config：

```
[IRI]
PORT = 14265
TCP_RECEIVER_PORT = 14600
UDP_RECEIVER_PORT = 14700
NEIGHBORS = udp://94.156.128.15:14600 udp://185.181.8.149:14600 udp://88.99.249.250:41041
IXI_DIR = ixi
HEADLESS = true
DEBUG = true
DB_PATH = db
```

上面的配置中，端口可以按照自己的要求进行修改，neighbours的内容是社区提供的可自动建立连接的一些节点。  

3.复制本地iri和配置文件到服务器：

```
scp ./iri-1.4.1.2.jar root@<your-ip>:~/iri/

scp ./iri.config root@<your-ip>:~/iri/
```

4.运行IRI

SSH到服务器，运行

```
screen -mSL iri java -jar iri-1.4.1.2.jar -c iri_config --remote --remote-limit-api "attachToTangle"
```

### 常用脚本

*查看节点信息：*

```
curl http://<your-ip>:<your-port> \
  -H 'X-IOTA-API-Version: 1' \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"command": "getNodeInfo"}'
```

*查看邻居节点连接状态：*

```
curl http://<your-ip>:<your-port> \
  -H 'X-IOTA-API-Version: 1' \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"command": "getNeighbors"}'
```

*添加邻居节点：*

```
curl http://<your-ip>:<your-port> \
  -X POST \
  -H 'X-IOTA-API-Version: 1' \
  -H 'Content-Type: application/json' \
  -d '{"command": "addNeighbors", "uris": ["tcp://<neighbour-ip>:<neighbour-port>"]}'
```

*删除邻居节点：*
```
curl http://<your-ip>:<your-port> \
  -X POST \
  -H 'X-IOTA-API-Version: 1' \
  -H 'Content-Type: application/json' \
  -d '{"command": "removeNeighbors", "uris": ["tcp://<neighbour-ip>:<neighbour-port>"]}'
```

### 节点同步

1. 想方设法加入[Slack](https://iotatangle.slack.com)，添加channel：nodesharing、mainnet-bootstrap和nelson。
寻找邻居节点，互粉之后一定做好记录，如果对方的节点中断，一定要及时告知，并清理无连接的节点。  
2. 可以选择社区提供的数据库来加速下载：https://iota.lukaseder.de/download.html   
下载完成之后替换自己的db文件，并修改对应数据库配置项。

### 节点管理
社区提供了开源的监控软件IOTA Peer Manager：
https://github.com/akashgoswami/ipm

### 自动发现的nelson

Nelson基于IRI的API，可以实现邻居节点的自动发现和连接，非常实用。参考Github地址：https://github.com/SemkoDev/nelson.cli

GUI监控如下图：
![nelson](/static/iri-setup/iri_aliyun_config.png)


## Reference
[1] https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04  
[2] https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/  
[3] https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-16-04  