---
title: 06-jwt原理
date: 2018-02-28 11:11:14
toc: true
tags: Spring Security
---



## 基于session认证所显露的问题

1. Session: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。

2. 扩展性: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

3. CSRF: 因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。

## 基于token的鉴权机制

基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。
流程上是这样的：

1. 用户使用用户名密码来请求服务器
2. 服务器进行验证用户的信息
3. 服务器通过验证发送给用户一个token
4. 客户端存储token，并在每次请求时附送上这个token值
5. 服务端验证token值，并返回数据


## 什么是JWT

Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。


## jwt组成：由.连接的三个字符串组成

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV
.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
##  第一部分：Header（头）
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

## 第二部分：payload（负载）
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

## 第三部分：signature（签名）
签名的过程：采用header中声明的算法，将 header部分用base64加密后的字符串、payload部分用base64加密后的字符串、秘钥盐（secret）用.连接起来组成的字符串，进行运算得到。这构成了jwt的第三部分。
```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```
secret是保存在服务器端的，jwt的签发也是在服务器端的，secret就是用来签发和验证jwt的。

## 具体Http请求
![image](img/jwt流程.png)

服务器端提供接口：
- 根据用户名和密码，为用户创建jwt返回
- 校验jwt是否合法,一般是在请求头里加入Authorization，并加上Bearer标注,服务端会获取并验证token，如果验证通过就会返回相应的资源

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

## jjwt使用

- dependency:
```
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.9</version>
</dependency>
```
- 生成jwt

```
Map map = new HashMap();
map.put("sub", "haha");
map.put("name", "fff");

String jws = Jwts.builder()
        .setClaims(map)
        .setExpiration(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2018-1-18 12:12:00"))
        .signWith(SignatureAlgorithm.HS512, "salt")
        .compact();
System.out.println(jws);

System.out.println(Jwts.parser().setSigningKey("salt").parseClaimsJws(jws).getBody().get("name"));
```
-  解析jwt

```
Claims getClaimsFromToken(String token) {
    Claims claims;
    try {
        claims = Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody();  
    } catch (Exception e) {
        claims = null;
    }
    return claims;
}
```



## 参考资料

> base64在线解码：https://www.base64decode.org/ 
> rfc7519的jwt规范：https://tools.ietf.org/html/rfc7519
> jjwt的github: https://github.com/jwtk/jjwt