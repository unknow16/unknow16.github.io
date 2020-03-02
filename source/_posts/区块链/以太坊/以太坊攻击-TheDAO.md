---
title: 以太坊攻击-The DAO
toc: true
date: 2019-12-26 15:54:38
tags:
categories:
---

## 什么是 The DAO 攻击

简单地讲，在2016年4月30日开始，一个名为“The DAO”的初创团队，在以太坊上通过[智能合约](https://learnblockchain.cn/2018/01/04/understanding-smart-contracts/)进行ICO众筹。28天时间，筹得1.5亿美元，成为历史上最大的众筹项目。

THE DAO创始人之一Stephan TualTual在6月12日宣布，他们发现了软件中存在的“递归调用漏洞”问题。 不幸的是，在程序员修复这一漏洞及其他问题的期间，一个不知名的黑客开始利用这一途径收集THE DAO代币销售中所得的以太币。6月18日，黑客成功挖到超过360万个以太币，并投入到一个DAO子组织中，这个组织和THE DAO有着同样的结构。

THE DAO持有近15%的以太币总数，因此THE DAO这次的问题对以太坊网络及其加密币都产生了负面影响。

6月17日，以太坊基金会的Vitalik Buterin更新一项重要报告，他表示，DAO正在遭到攻击，不过他已经研究出了解决方案：

现在提出了软件分叉解决方案，通过这种软件分叉，任何调用代码或委托调用的交易——借助代码hash0x7278d050619a624f84f51987149ddb439cdaadfba5966f7cfaea7ad44340a4ba（也就是DAO和子DAO）来减少账户余额——都会视为无效……

最终因为社交的不同意见，最终以太坊分裂出支持继续维持原状的以太经典 ETC，同意软件分叉解决方案的在以太坊当前网络实施。

> 以上内容整理自文章[The DAO 攻击](http://chainb.com/?P=Cont&id=1290)。

## 解决方案

因为投资者已经将以太币投入了 The DAO 合约或者其子合约中，在攻击后无法立刻撤回。
需要让投资者快速撤回投资，且能封锁黑客转移资产。

V神公布的解决方案是，在程序中植入转移合约以太币代码，让矿工选择是否支持分叉。在分叉点到达时则将 The DAO 和其子合约中的以太币转移到一个新的安全的可取款合约中。全部转移后，原投资者则可以直接从取款合约中快速的拿回以太币。取款合约在讨论方案时，已经部署到主网。合约地址是[0xbf4ed7b27f1d666546e30d74d50d173d20bca754](https://[etherscan]%28https//learnblockchain.cn/docs/etherscan/).io/address/0xbf4ed7b27f1d666546e30d74d50d173d20bca754)。



取款合约代码如下：

```
// Deployed on mainnet at 0xbf4ed7b27f1d666546e30d74d50d173d20bca754
contract DAO {
    function balanceOf(address addr) returns (uint);
    function transferFrom(address from, address to, uint balance) returns (bool);
    uint public totalSupply;
}
contract WithdrawDAO {
    DAO constant public mainDAO = DAO(0xbb9bc244d798123fde783fcc1c72d3bb8c189413);
    address public trustee = 0xda4a4626d3e16e094de3225a751aab7128e96526;
    function withdraw(){
        uint balance = mainDAO.balanceOf(msg.sender);
        if (!mainDAO.transferFrom(msg.sender, this, balance) || !msg.sender.send(balance))
            throw;
    }
    function trusteeWithdraw() {
        trustee.send((this.balance + mainDAO.balanceOf(this)) - mainDAO.totalSupply());
    }
}
```

同时，为照顾两个阵营，软件提供硬分叉开关，选择权则交给社区。支持分叉的矿工会在X区块到X+9区块出块时，在区块`extradata`字段中写入`0x64616f2d686172642d666f726b`（“dao-hard-fork”的十六进制数）。从分叉点开始，如果连续10个区块均有硬分叉投票，则表示硬分叉成功。



但是在当前版本中，社区已完成硬分叉，所以已移除开关类代码。

当前，主网已默认配置支持DAO分叉，并设定了开始硬分叉高度 1920000，代码如下：

```
// params/config.go:38
MainnetChainConfig = &ChainConfig{ 
        DAOForkBlock:        big.NewInt(1920000),
        DAOForkSupport:      true, 
    }
```

## 参考资料
> - []()
> - []()
