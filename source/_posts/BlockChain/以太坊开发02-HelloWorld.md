---
title: 以太坊开发02--HelloWorld
date: 2018-04-19 19:43:12
tags: BlockChain
---

### Solidity安装
可以先采用[Browser-Solidity](https://ethereum.github.io/browser-solidity)来进行开发。

### Geth安装
1. 新安装的Ubuntu16.04,没有安装ssh-server,需要安装后才能用远程连接
```
> sudo apt-get install openssh-server
```
2. Geth安装

```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo add-apt-repository -y ppa:ethereum/ethereum-dev
sudo apt-get update
sudo apt-get install ethereum
```
3. 测试成功

```
geth --help
```



### 启动环境
```
//启动一个以太坊网络节点。
geth --datadir testNet --dev console 2>> test.log

//参数说明：
--dev 启用开发者网络（模式），开发者网络会使用POA共识，默认预分配一个开发者账户并且会自动开启挖矿。
--datadir 后面的参数是区块数据及密钥存放目录。第一次输入命令后，它会放在当前目录下新建一个testNet目录来存放数据。

console  进入控制台

2>>test.log  表示把控制台日志输出到test.log文件

//可以新开一个终端，实时显示日志
tail -f test.log
```

### 准备账户
部署智能合约需要一个外部账户，--dev参数会默认分配一个开发者账户。

```
//查看账户,两种方式
eth.accounts
personal.listAccounts

//查看账户余额
eth.getBalance(eth.accounts[0])

//新建账户
personal.newAccount("fuyi"); 
//fuyi为密码，解锁账户时要用，余额为0

//给新建账户转钱
eth.sendTransaction({from: '', to: '', value: web3.toWei(1, "ether")})

//解锁账户
personal.unlockAccount(eth.account[1], "fuyi");
//部署合约前要先解锁账户，即验证密码,fuyi即创建账户时的密码
```

### 编写合约代码

```
pragma solidity ^0.4.18;

contract hello {
    string greeting;
    
    function hello(string _greeting) public {
        greeting = _greeting;
    }

    function say() constant public returns (string) {
        return greeting;
    }
}
```
简单解释下，我们定义了一个名为hello的合约，在合约初始化时保存了一个字符串（我们会传入hello world），每次调用say返回字符串。
把这段代码写(拷贝)到Browser-Solidity，如果没有错误，点击Details获取部署代码.

在弹出的对话框中找到WEB3DEPLOY部分，点拷贝，粘贴到编辑器后，修改初始化字符串为hello world。

### 部署合约
将上步的代码拷贝回geth控制台里，回车后，看到输出如：
```
Contract mined! address: 0x79544078dcd9d560ec3f6eff0af42a9fc84c7d19 transactionHash: 0xe2caab22102e93434888a0b8013a7ae7e804b132e4a8bfd2318356f6cf0480b3
```
说明合约已经部署成功。

* 此时
1. 在打开的tail -f test.log日志终端里，可以同时看到挖矿记录
2. 现在我们查看下新账户的余额应该是比开始少了

### 运行合约

```
//可看到合约信息
> hello

//调用函数
> hello.say()
```

