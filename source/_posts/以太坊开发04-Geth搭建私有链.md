---
title: 以太坊开发04--Geth搭建私有链
date: 2018-04-19 19:44:03
tags: BlockChain
---

### Geth客户端安装
Geth,它是一个命令行界面，执行在Go上实现的完整的以太坊节点。Geth得益于Go语言的多平台特性，支持在多个平台上使用，比如：win、linux、mac等。

Geth是以太坊协议的具体落地实现，通过其可以实现以太坊的各种功能，如：账户创建编辑删除、开启挖矿、ether币的转移、智能合约的部署和执行等。

```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum

// 获取geth指令
geth --help
```

### 准备创世区块信息
以太坊支持自定义创世区块，要运行私有链，我们就需要定义自己的创世区块，创世区块信息写在一个json格式的配置文件中。首先将下面的内容保存到一个json文件中，例如genesis.json。
json文件内容如下:
```
{
  "config": {
        "chainId": 10, 
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

### 私有链初始化：写入创始区块
准备好创世区块json配置文件后，需要初始化区块链，将上面的创世区块信息写入到区块链中。首先要新建一个目录data0用来存放区块链数据(其实，这个目录data0就相当于一个根节点。当我们基于genesis.json生成根节点后，其他人就可以来连接此根节点，从而能进行交易)。


```
//1. 创建一个空文件夹
> mkdir privatechain
> cd privatechain

//2. 将genesis.json放到privatechain下

//3. 初始化
> geth --datadir data0 init genesis.json

```
上面的命令的主体是 geth init，表示初始化区块链，命令可以带有选项和参数，其中–datadir选项后面跟一个目录名，这里为 data0，表示指定数据存放目录为 data0， genesis.json是init命令的参数。

运行上面的命令，会读取genesis.json文件，根据其中的内容，将创世区块写入到区块链中。如果看到log信息中含有Successfully wrote genesis state字样，说明初始化成功。

其中data0/geth/chaindata中存放的是区块数据，data0/keystore中存放的是账户数据。

### 启动私有链节点
初始化完成后，就有了一条自己的私有链，之后就可以启动自己的私有链节点并做一些操作，在终端中输入以下命令即可启动节点：
```
geth --datadir data0 --networkid 1108 console
```
上面命令的主体是geth console，表示启动节点并进入交互式控制台，–datadir选项指定使用data0作为数据目录，–networkid选项后面跟一个数字，这里是1108，表示指定这个私有链的网络id为1108。网络id在连接到其他节点的时候会用到，以太坊公网的网络id是1，为了不与公有链网络冲突，运行私有链节点的时候要指定自己的网络id。

运行上面的命令后，就启动了区块链节点并进入了Javascript Console。这是一个交互式的Javascript执行环境，在这里面可以执行Javascript代码，其中>是命令提示符。在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。这些对象主要包括：

- eth：包含一些跟操作区块链相关的方法
- net：包含以下查看p2p网络状态的方法
- admin：包含一些与管理节点相关的方法
- miner：包含启动&停止挖矿的一些方法
- personal：主要包含一些管理账户的方法
- txpool：包含一些查看交易内存池的方法
- web3：包含了以上对象，还包含一些单位换算的方法

##### 下节具体介绍上面对象的用法