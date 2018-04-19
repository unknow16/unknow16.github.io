---
title: Ubuntu16.04-安装Geth
date: 2018-04-19 20:02:53
tags: Linux
---

### git安装
之后的安装都需要依赖Git

```
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git

// 查看版本号
git --version
git version 2.10.2
```

### geth

```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum

// 获取geth指令
geth --help
```

### solc安装
solidity是以太坊智能合约的开发语言。想要测试智能合约，开发DAPP的需要安装solc。

```
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```
