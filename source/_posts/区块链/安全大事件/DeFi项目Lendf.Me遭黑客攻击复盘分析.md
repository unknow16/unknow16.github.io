---
title: DeFi项目Lendf.Me遭黑客攻击复盘分析
toc: true
date: 2020-12-31 13:06:27
tags:
categories:
---

2020年4月18日08:58，黑客利用 Uniswap 和 ERC777 的兼容性问题，在进行 ETH-imBTC 交易时，利用 ERC777 中的多次迭代调用 tokensToSend 来实现重入攻击，将 Uniswap 上的 imBTC（imBTC 是一个 1:1 锚定比特币的 ERC-20 代币）池耗尽。

2020年4月19日09:28，Lendf.me 遭遇类似 Uniswap 事件的重入攻击，出现大量异常借贷行为，攻击者利用重入漏洞覆盖自己的资金余额并使得可提现的资金量不断翻倍，黑客以滚雪球的方式多笔转走 imBTC，且每一笔都比上一笔翻倍，最终将 Lendf.Me 账户资产盗取一空。

此次黑客攻击，共累计的损失约 24,696,616 美元，攻击成功后，攻击者不断通过 OneInchExchange、AugustusSwapper、Tokenlon 等去中心化交易平台以及 Compound 借贷平台将盗取的币进行兑换和转移。

本次针对 Lendf.Me 实施攻击的攻击者地址为： 0xa9bf70a420d364e923c74448d9d817d3f2a77822，攻击者的合约地址为： 0x538359785a8d5ab1a741a0ba94f26a800759d91d ，攻击者通过此合约对 Lendf.Me 进行攻击。

## 参考资料
> - []()
> - []()
