---
title: 08-Http常见认证方式
toc: true
date: 2019-07-26 11:20:57
tags:
categories:
---



## 常用编码方式

1. URLEncode

url编码主要是为了解决一些url中的一些特殊字符和歧义字符或者中文字符的传输问题

* 编码（URLEnCode）：

数字和字母不变，中文会变化。
空格变为"+"号。
其他被编码成"%"加上他们的ascii的十六进制，规律是这样的。

* 解码（URLDeCode）：

如果是页面解码，其实Request.QueryString()会自动做解码的动作。无需再写一遍
如果是其他地方调用，如Andriod调用.net的WebService，则需要做一次解码的动作

2.  Base64编码
- 理解成可逆加密算法，只能防肉眼看到真实内容
- Base64的长度是有限制的
- 配合加密使用，建议先对文本做加密处理，在最外面再做Base64处理。

3.  SHA1/MD5
- 可以理解成不可逆加密算法



## 基本认证（Basic Authentication）

基本认证是Http1.0提出的认证方法。将认证信息放在请求头 Authorization中，值为Basic xxx，注意Basic和xxx中间有一个空格，xxx是把 "用户名+冒号+密码"用BASE64算法加密后的字符串。

每次客户端请求都需带上Authorization请求头。若客户端是浏览器，则浏览器会提供一个输入用户名和密码的对话框，用户输入用户名和密码后，浏览器会保存用户名和密码，用于构造Authorization值。当关闭浏览器后，用户名和密码将不再保存。

用户名/密码经过Base64加码后，这个Base64码值可以轻易被解码并获得用户名/密码，所以此认证方式并不安全。为了传输安全，需要配合SSL使用。

## 令牌认证（Bearer Token）
Bearer Token (RFC 6750) 用于OAuth 2.0授权访问资源，任何Bearer持有者都可以无差别地用它来访问相关的资源，而无需证明持有加密key。一个Bearer代表授权范围、有效期，以及其他授权事项；一个Bearer在存储和传输过程中应当防止泄露，需实现Transport Layer Security (TLS)；一个Bearer有效期不能过长，过期后可用Refresh Token申请更新。



#### 请求资源

Bearer实现资源请求有三种方式：Authorization Header、Form-Encoded Body Parameter、URI Query Parameter，这三种方式优先级依次递减

- Authorization Header：该头部定义与Basic方案类似
```
GET /resource HTTP/1.1
Host: server.example.com
Authorization: Bearer mF_9.B5f-4.1JqM

```
-  Form-Encoded Body Parameter： 下面是用法实例
```
POST /resource HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

access_token=mF_9.B5f-4.1JqM
```
使用该方法发送Bearer须满足如下条件：

1. 头部必须包含"Content-Type: application/x-www-form-urlencoded"
2. entity-body必须遵循application/x-www-form-urlencoded编码(RFC 6749)
3. 如果entity-body除了access_token之外，还包含其他参数，须以"&"分隔开
4. entity-body只包含ASCII字符
5. 要使用request-body已经定义的请求方法，不能使用GET

如果客户端无法使用Authorization请求头，才应该使用该方法发送Bearer

- URI Query Parameter：
```
GET /resource?access_token=mF_9.B5f-4.1JqM HTTP/1.1
Host: server.example.com
Cache-Control: no-store
```
服务端应在响应中使用 Cache-Control: private

#### WWW-Authenticate响应头

在客户端未发送有效Bearer的情况下，即错误发生时，资源服务器须发送WWW-Authenticate头，下为示例：

  ```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="example", error="invalid_token", error_description="The access token expired"
  ```

下面将就WWW-Authenticate字段的用法进行详细描述(下列这些属性/指令不应重复使用)：

- **Bearer**：Beare作为一种认证类型(基于OAuth 2.0)，使用"Bearer"关键词进行定义

- **realm**：与Basic、Digest一样，Bearer也使用相同含义的域定义reaml

- **scope**：授权范围，可选的，大小写敏感的，空格分隔的列表(%x21 / %x23-5B / %x5D-7E)，可以是授权服务器定义的任何值，不应展示给终端用户。OAuth 2.0还规定客户端发送scope请求参数以指定授权访问范围，而在实际授权范围与客户端请求授权范围不一致时，授权服务器可发送scope响应参数以告知客户端下发的token实际的授权范围。下为两个scope用法实例：

  ```
  scope="openid profile email"
  scope="urn:example:channel=HBO&urn:example:rating=G,PG-13"
  ```

- **error**：描述访问请求被拒绝的原因，字符%x20-21 / %x23-5B / %x5D-7E之内

- **error_description**：向开发者提供一个可读的解释，字符%x20-21 / %x23-5B / %x5D-7E之内

- **error_uri**：absolute URI，标识人工可读解释错误的页面，字符%x21 / %x23-5B / %x5D-7E之内

　　当错误发生时，资源服务器将发送的HTTP Status Code(通常是400, 401, 403, 或405)及Error Code如下：

- **invalid_request**：请求丢失参数，或包含无效参数、值，参数重复，多种方法发送access token，畸形等。资源服务器将发送HTTP 400 (Bad Request)
- **invalid_token**：access token过期、废除、畸形，或存在其他无效理由的情况。资源服务器将发送HTTP 401 (Unauthorized)，而客户端则需要申请一个新的access token，然后才能重新发送该资源请求
- **insufficient_scope**：客户端提供的access token的享有的权限太低。资源服务器将发送HTTP 403 (Forbidden)，同时WWW-Authenticate头包含scope属性，以指明需要的权限范围

　　如果客户端发送的资源请求缺乏任何认证信息(如缺少access token，或者使用 [RFC 6750](http://www.rfcreader.com/#rfc6750) 所规定的三种资源请求方式之外的任何method)，资源服务器不应该在响应中包含错误码或者其他错误信息，如下即可：

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="example"
```

#### Bearer Token Response

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
   "access_token":"mF_9.B5f-4.1JqM",
   "token_type":"Bearer",
   "expires_in":3600,
   "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA"
}
```



## **MAC** **Token**

　MAC Token与Bearer Token一样，可作为OAuth 2.0的一种Access Token类型，但Bearer Token才是RFC建议的标准；MAC Token是在[MAC Access Authentication](https://tools.ietf.org/html/draft-hammer-oauth-v2-mac-token-05)中被定义的，采用Message Authentication Code(MAC)算法来提供完整性校验。

[MAC Access Authentication](https://tools.ietf.org/html/draft-hammer-oauth-v2-mac-token-05) 是一种HTTP认证方案，具体内容可参考： [HTTP Authentication: MAC Access Authentication draft-hammer-oauth-v2-mac-token-05](https://tools.ietf.org/html/draft-hammer-oauth-v2-mac-token-05) 。



## 摘要认证 （digest authentication）

Http1.1提出的基本认证的替代方法。

##  WSSE(WS-Security)认证

扩展HTTP认证


## 参考资料
> - []()
> - []()
