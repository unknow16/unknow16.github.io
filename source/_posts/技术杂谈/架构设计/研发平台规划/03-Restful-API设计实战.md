---
title: 03-Restful API设计实战
toc: true
date: 2019-08-16 10:21:00
tags:
categories:
---



## 前言

RESTful是一种设计模式，或者说是一种设计规范，它并没有太多强制性的要求之类的，实际上它有的只是几个原则，当一个应用满足这些原则的时候，可以认为它是RESTful的。

这些规范的具体内容不再详述。RESTful可以认为是一种建立在HTTP协议之上的设计模式，充分的利用了HTTP协议的特定，使用URL来表示资源，用各个不同的HTTP动词来表示对资源的各种行为。

> 这样做的好处就是资源和操作分离，让资源的管理更加规范。

RESTful API一般采用OAuth 2.0协议来进行身份认证和访问权限控制，其中的资源就对应了OAuth中的资源服务器的资源。

## URL格式设计

1. 域名： 应该尽量将API部署在专用域名之下

```
https://api.example.com
```

2. 版本：应该将API的版本号放入URL

```
https://api.example.com/v1/
```

3. 路径/端点： 表示API的具体地址

```
GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID：删除某个动物园

// 如下两者等同,推荐后者
GET /zoos/ID/animals：列出某个指定动物园的所有动物
GET /animals?zoo_id=ID

// 如下两者等同,推荐后者
DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
DELETE /animals/ID?zoo_id=ID
```

4. 过滤条件

```
?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1：指定筛选条件
```



## 响应状态码

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

状态码的完全列表参见 https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html



## 方法覆盖

有些客户端只能使用`GET`和`POST`这两种方法。服务器必须接受`POST`模拟其他三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上`X-HTTP-Method-Override`属性，告诉服务器应该使用哪一个动词，覆盖`POST`方法。

```
POST /api/Person/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```

上面代码中，`X-HTTP-Method-Override`指定本次请求的方法是`PUT`，而不是`POST`。




## HATEOAS

API 的使用者未必知道，URL 是怎么设计的。一个解决方法就是，在回应中，给出相关链接，便于下一步操作。这样的话，用户只要记住一个 URL，就可以发现其他的 URL。这种方法叫做 HATEOAS，这种API叫做Hypermedia API。Github的API就是这种设计，访问[api.github.com](https://api.github.com/)会得到一个所有可用API的网址列表。





## 参考资料
> - []()
> - []()
