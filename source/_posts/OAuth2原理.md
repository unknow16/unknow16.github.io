---
title: OAuth2原理
date: 2018-02-28 13:27:29
tags: Spring Security
---

### 场景
* 角色： 云冲印应用， Google照片云盘，用户
* 需求：用户想使用云冲印应用冲印照片，需要云冲印应用访问自己在Google中的照片。
* 做法：用户授权云冲印应用去访问自己访问Google中照片的权限。

### 名词解释
- （1） Third-party application：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"。
- （2）HTTP service：HTTP服务提供商，本文中简称"服务提供商"，即上一节例子中的Google。
- （3）Resource Owner：资源所有者，本文中又称"用户"（user）。
- （4）User Agent：用户代理，本文中就是指浏览器。
- （5）Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。
- （6）Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

### OAuth的思路
OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。
"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

![image](https://note.youdao.com/yws/api/personal/file/3F5D4F45228949D99E9BCB9D6CDD62FD?method=download&shareKey=aa3cfc804bb7645db7d78c54e9f2f4a2)

- （A）用户打开客户端以后，客户端要求用户给予授权。
- （B）用户同意给予客户端授权。
- （C）客户端使用上一步获得的授权，向认证服务器申请令牌。
- （D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
- （E）客户端使用令牌，向资源服务器申请获取资源。
- （F）资源服务器确认令牌无误，同意向客户端开放资源。

不难看出来，上面六个步骤之中，B是关键，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。
下面一一讲解客户端获取授权的四种模式。

### 客户端获取授权的四种模式
- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

### 密码模式
![image](https://note.youdao.com/yws/api/personal/file/BFB659BC33C94F5BB6B2BA532277F95D?method=download&shareKey=d71f118c94869a848487ec3c50039bef)

- 用户在  有授权服务器的应用中   注册的有账号
- client   已开发支持   应用的授权接入

目标：client要获取用户在其他应用中的资源，其它应用需有OAuth授权服务器和资源服务器

A: client请求获取用户在其它应用中的用户名密码
B: client请求其他应用的授权服务器，获取token
C:其他应用返回token
结果： client可用该token去其他应用资源服务器获取资源

### 客户端模式：client自已作为用户
![image](https://note.youdao.com/yws/api/personal/file/D004E6A427C04066906F0D070447B7B4?method=download&shareKey=28d93837b5bcb382a3b937cfc0cc7411)

### 授权码模式
![image](https://note.youdao.com/yws/api/personal/file/29905A687A62407AA7463825A11364F6?method=download&shareKey=220b0268b259ca5be55e6f5e3deb30a1)
- （A）用户访问客户端，后者将前者导向认证服务器。
- （B）用户选择是否给予客户端授权。
- （C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
- （D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
- （E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

阮老师：http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
