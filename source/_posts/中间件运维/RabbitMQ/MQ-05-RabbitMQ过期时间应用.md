---
title: MQ-05-RabbitMQ过期时间应用
date: 2018-10-08 19:05:22
tags: MQ
---


## TTL
TTL是time to live 的简称，顾名思义指的是消息的存活时间。rabbitMq可以从两种维度设置消息过期时间，分别是队列和消息本身。

注意当队列过期时间和消息过期时间都存在时，取两者中较短的时间。

## 设置队列中所有消息的过期时间
通过设置队列的x-message-ttl参数来设置指定队列上消息的存活时间，其值是一个非负整数，单位为毫秒秒。不同队列的过期时间互相之间没有影响，即使是对于同一条消息。

队列中的消息存在队列中的时间超过过期时间则成为死信。
```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 60000); // 60秒
channel.queueDeclare("myqueue", false, false, false, args);

```

## 设置单独消息的过期时间
发送消息时可以单独指定每条消息的TTL，通过设置AMQP.Properties的expiration属性可以指定消息的过期时间。 

```
byte[] messageBodyBytes = "Hello, world!".getBytes();
AMQP.BasicProperties properties = new AMQP.BasicProperties();
properties.setExpiration("60000");
channel.basicPublish("my-exchange", "routing-key", properties, messageBodyBytes);
```

## 死信交换机DLX
队列中的消息在以下三种情况下会变成死信 
1. 消息被拒绝（basic.reject 或者 basic.nack），并且requeue=false
2. 消息的过期时间到期了
3. 队列长度限制超过了

当队列中的消息成为死信以后，如果队列设置了DLX那么消息会被发送到DLX。

通过x-dead-letter-exchange设置DLX，通过这个x-dead-letter-routing-key设置消息发送到DLX所用的routing-key，如果不设置默认使用消息本身的routing-key;
```
channel.exchangeDeclare("some.exchange.name", "direct");

Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "some.exchange.name");
args.put("x-dead-letter-routing-key", "some-routing-key");
channel.queueDeclare("myqueue", false, false, false, args);
```

## 延时队列
现在我们知道了，我们既可以控制消息在一段时间后变成死信，又可以控制变成死信的消息被路由到某一个指定的交换机，结合二者，其实就可以实现一个延时队列。消息的过期时间又两种设置方法，延时队列对应的也有两种实现思路。如下图：

![image](https://note.youdao.com/yws/api/personal/file/6E7E37807F0345DA9B731DE483E84202?method=download&shareKey=cc0dd0c54eea2b10d598246f5d7cd67d)