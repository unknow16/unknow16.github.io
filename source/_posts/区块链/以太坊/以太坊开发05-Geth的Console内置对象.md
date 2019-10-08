---
title: 以太坊开发05--Geth的Console内置对象
date: 2018-04-19 19:44:17
tags: BlockChain
---

Geth的Console是一个交互式的Javascript执行环境，在这里面可以执行Javascript代码，其中>是命令提示符。

在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。这些对象主要包括：

- eth：包含一些跟操作区块链相关的方法
- net：包含以下查看p2p网络状态的方法
- admin：包含一些与管理节点相关的方法
- miner：包含启动&停止挖矿的一些方法
- personal：主要包含一些管理账户的方法
- txpool：包含一些查看交易内存池的方法
- web3：包含了以上对象，还包含一些单位换算的方法


### 账户管理
* 创建账户

如下：Passphrase其实就是密码的意思，输入两次密码后，就创建了一个账户.
```
> personal.newAccount()
> Passphrase:
> Repeat passphrase:
"0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f"
```
再次执行，则会又创建了一个新账户。

* 查看账户

```

> eth.accounts
["0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f", "0x46b24d04105551498587e3c6ce2c3341d5988938"]
```

账户默认会保存在数据目录的keystore文件夹中。查看目录结构，发现data0/keystore中多了两个文件，这两个文件就对应刚才创建的两个账户，这是json格式的文本文件，可以打开查看，里面存的是私钥经过密码加密后的信息。

* 查看账户余额

```
> eth.getBalance(eth.accounts[0])
0
> eth.getBalance(eth.accounts[1])
0
```
目前两个账户的以太币余额都是0，要使账户有余额，可以从其他账户转账过来，或者通过挖矿来获得以太币奖励。

### 挖矿
* 启动挖矿

```
> miner.start(10);
```
其中start的参数表示挖矿使用的线程数。第一次启动挖矿会先生成挖矿所需的DAG文件，这个过程有点慢，等进度达到100%后，就会开始挖矿，此时屏幕会被挖矿信息刷屏。

* 停止挖矿

如果想停止挖矿，并且进度已经达到100%之后，可以在js console中输入
```
> miner.stop();
```
注意：输入的字符会被挖矿刷屏信息冲掉，没有关系，只要输入完整的miner.stop()之后回车，即可停止挖矿。

* 奖励账户

挖到一个区块会奖励5个以太币，挖矿所得的奖励会进入矿工的账户，这个账户叫做coinbase，默认情况下coinbase是本地账户中的第一个账户：
```
> eth.coinbase
"0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f"
```
现在的coinbase是账户0，要想使挖矿奖励进入其他账户，通过miner.setEtherbase()将其他账户设置成coinbase即可：
```
> miner.setEtherbase(eth.accounts[1])
true
> eth.coinbase
"0x46b24d04105551498587e3c6ce2c3341d5988938"

```
挖到区块以后，账户0里面应该就有余额了：
```
> eth.getBalance(eth.accounts[0])
2.31e+21
```
getBalance()返回值的单位是wei，wei是以太币的最小单位，1个以太币=10的18次方个wei。要查看有多少个以太币，可以用web3.fromWei()将返回值换算成以太币：
```
> web3.fromWei(eth.getBalance(eth.accounts[0]),'ether')
2310
```

### 交易
截止目前，账户一的余额还是0

可以通过发送一笔交易，从账户0转移10个以太币到账户1：
```
> amount = web3.toWei(10,'ether')
"10000000000000000000"
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})
Error: authentication needed: password or unlock
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:1
```
这里报错了，原因是账户每隔一段时间就会被锁住，要发送交易，必须先解锁账户，由于我们要从账户0发送交易，所以要解锁账户0：
```
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f
Passphrase: 
true
```
输入创建账户时设置的密码，就可以成功解锁账户。然后再发送交易：
```
> amount = web3.toWei(10,'ether')
"10000000000000000000"
> eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})
INFO [03-07|11:13:11] Submitted transaction                    fullhash=0x1b21bba16dd79b659c83594b0c41de42debb2738b447f6b24e133d51149ae2a6 recipient=0x46B24d04105551498587e3C6CE2c3341d5988938
"0x1b21bba16dd79b659c83594b0c41de42debb2738b447f6b24e133d51149ae2a6"
```
我们去查看账户1中的余额,发现还没转过去，此时交易已经提交到区块链，但还未被处理，这可以通过查看txpool来验证：
```
> txpool.status
{
  pending: 1,
  queued: 0
}
```
其中有一条pending的交易，pending表示已提交但还未被处理的交易。

要使交易被处理，必须要挖矿。这里我们启动挖矿，然后等待挖到一个区块之后就停止挖矿：
```
> miner.start(1);admin.sleepBlocks(1);miner.stop();
```
当miner.stop()返回true后，txpool中pending的交易数量应该为0了，说明交易已经被处理了，而账户1应该收到币了.

### 查看交易和区块
* 查看当前区块总数：
```
> eth.blockNumber
463
```
* 通过区块号查看区块

```
> eth.getBlock(66)
{
  difficulty: 135266,
  extraData: "0xd783010802846765746886676f312e31308664617277696e",
  gasLimit: 3350537,
  gasUsed: 0,
  hash: "0x265dfcc0649bf6240812256b2b9b4e3ae48d51fd8e43e25329ac111556eacdc8",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f",
  mixHash: "0xaf755722f62cac9b483d3437dbc795f2d3a02e28ec03d39d8ecbb6012906263c",
  nonce: "0x3cd80f6ec5c2f3e9",
  number: 66,
  parentHash: "0x099776a52223b892d13266bb3aec3cc04c455dc797185f0b3300d39f9fc0a8ec",
  receiptsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 535,
  stateRoot: "0x0c9feec5a201c8c98618331aecbfd2d4d93da1c6064abd0c41ae649fc08d8d06",
  timestamp: 1520391527,
  totalDifficulty: 8919666,
  transactions: [],
  transactionsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  uncles: []
}
```
* 通过交易hash查看交易

```
> eth.getTransaction("0x1b21bba16dd79b659c83594b0c41de42debb2738b447f6b24e133d51149ae2a6")
{
  blockHash: "0x1cb368a27cc23c786ff5cdf7cd4351d48f4c8e8aea2e084a5e9d7c480449c79a",
  blockNumber: 463,
  from: "0x4a3b0216e1644c1bbabda527a6da7fc5d178b58f",
  gas: 90000,
  gasPrice: 18000000000,
  hash: "0x1b21bba16dd79b659c83594b0c41de42debb2738b447f6b24e133d51149ae2a6",
  input: "0x",
  nonce: 0,
  r: "0x31d22686e0d408a16497becf6d47fbfdffe6692d91727e5b7ed3d73ede9e66ea",
  s: "0x7ff7c14a20991e2dfdb813c2237b08a5611c8c8cb3c2dcb03a55ed282ce4d9c3",
  to: "0x46b24d04105551498587e3c6ce2c3341d5988938",
  transactionIndex: 0,
  v: "0x38",
  value: 10000000000000000000
}
```
