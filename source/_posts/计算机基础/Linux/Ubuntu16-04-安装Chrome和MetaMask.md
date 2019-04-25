---
title: Ubuntu16.04-安装Chrome和MetaMask
date: 2018-04-19 20:06:23
tags: Linux
---

### 安装Chrome

### Chrome安装MetaMask插件
MetaMask 是一款Chrome插件形式的以太坊轻客户端，开发过程中使用MetaMask和我们的dapp进行交互是个很好的选择。

1. MetaMask安装

MetaMask插件和安装一般Chrome插件无区别。安装完成后，浏览器工具条会显示一个小狐狸图标。

2. 配置钱包

接受隐私后，点击页面中的Import Existing DEN,输入Ganache显示的助记词。

再输入自己想要的钱包密码，点击OK.

3. 连接开发私链网络（如下连Ganache）

默认连接的是以太坊主网（左上角显示），选择Custom RPC，添加一个网络：http://127.0.0.1:7545


这时左上角显示为Private Network，账号是Ganache中默认的第一个账号。

4. 至此MetaMask的安装，配置已经完成。