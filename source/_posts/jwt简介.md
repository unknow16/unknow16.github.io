---
title: jwt原理
date: 2018-02-28 11:11:14
tags: Spring Security
---

* jwt详解：https://www.jianshu.com/p/576dbf44b2ae
* base64在线解码：https://www.base64decode.org/ 
* rfc7519的jwt规范：https://tools.ietf.org/html/rfc7519
* http://blog.csdn.net/sxdtzhaoxinguo/article/details/77965226

![image](https://note.youdao.com/yws/api/personal/file/F4B2C8FD47374D76B58A69F6A27373A7?method=download&shareKey=bb536cf845bc080a447a27dc0332fbfe)

一般是在请求头里加入Authorization，并加上Bearer标注：
服务端会验证token，如果验证通过就会返回相应的资源。

### jwt组成：由.连接的三个字符串组成
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV
.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
### 第一部分：Header（头）
通常为
```
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```
typ声明类型，目前值业界都写JWT，可省略
alg声明加密的算法，通常为HMAC SHA256

将该json用base64编码加密，得到jwt的第一部分

### 第二部分：payload（负载）
payload中可以放置三类数据：系统保留的、公共的和私有的：
通常为
```
{
    "sub":"wang",
    "created":1489079981393,
    "exp":1489684781
}
```

这段告诉我们这个Token中含有的数据声明（Claim），这个例子里面有三个声明：sub, created 和 exp。在我们这个例子中，分别代表着用户名、创建时间和过期时间，当然你可以把任意数据声明在这里。

将该json用base64编码加密，得到jwt的第二部分

### 第三部分：signature（签名）
签名的过程：采用header中声明的算法，将 header部分用base64加密后的字符串、payload部分用base64加密后的字符串、秘钥盐（secret）用.连接起来组成的字符串，进行运算得到。这构成了jwt的第三部分。
```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```
secret是保存在服务器端的，jwt的签发也是在服务器端的，secret就是用来签发和验证jwt的。

### 具体Http请求
![image](https://note.youdao.com/yws/api/personal/file/D08E456040724A6E91C3A7151AEF7C64?method=download&shareKey=99d17c82adb574c226713cdbaf185dfc)

服务器端提供接口：
- 根据用户名和密码，为用户创建jwt返回
- 校验jwt是否合法

1，2，3

```
POST 

http://localhost:8080/auth

Content-Type: application/json

{"username":"1234","password":"1234"}

---------------------------------------
结果

{
  "token" : "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxMjM0IiwiY3JlYXRlZCI6MTUwMzQxMzMwODkxOCwiZXhwIjoxNTA0MDE4MTA4fQ.jQc5MRdgKfi5ds1N0ZSsxkunQQVkFuGJ7Giv1_JrjTiKsu3h7UwE8vjU5wVPaipM_zkbHaMpRqXvF__ci5p7aw"
}
```
4，5，6

```
GET

http://localhost:8080/user-service/bizUser/getUserScore

Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxMjM0IiwiY3JlYXRlZCI6MTUwMzQxMzMwODkxOCwiZXhwIjoxNTA0MDE4MTA4fQ.jQc5MRdgKfi5ds1N0ZSsxkunQQVkFuGJ7Giv1_JrjTiKsu3h7UwE8vjU5wVPaipM_zkbHaMpRqXvF__ci5p7aw

---------------------------------------
结果
[
  {
    "id": 11,
    "username": "123",
    "password": "456",
    "scoreList": [
      {
        "id": 1,
        "score": 100
      }
    ]
  }
]
```
认证失败时

```
{
  "timestamp": 1503413947608,
  "status": 401,
  "error": "Unauthorized",
  "message": "手动滑稽(　 ´-ω ･)▄︻┻┳══━一",
  "path": "/user-service/bizUser/getUserScore"
}
```
