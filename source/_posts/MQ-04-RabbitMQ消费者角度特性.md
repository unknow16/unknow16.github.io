---
title: MQ-04-RabbitMQ消费者角度特性
date: 2018-10-08 19:04:39
tags: MQ
---

生产者角度一篇从生产者角度介绍如何保证消息被正确发送到服务器，如果未正确发送如何处理；本篇博客将从消费者角度介绍三个问题：队列分发消息到消费者的规则、如何确保消息一定被正确接受并处理了、如何保证多个消费者负载均衡。

## 消息分发规则
官网的示例中介绍了默认情况下rabbitMqRabbitMQ会一个一个的发送信息给下一个消费者(consumer)，而不考虑每个任务的时长等等，且是一次性分配，并非一个一个分配。平均的每个消费者将会获得相等数量的消息。这样分发消息的方式叫做round-robin。

## 消息应答模式
默认情况下消费者C1接收到消息1无论是否正常接受和处理都会立即应答rabbit服务器，然后消息1就会从队列中被删除，假如C1突然出现异常状况导致消息1没有被处理完毕，那么消息1就处理失败了，也不会有其他消费者去处理消息1。事实上我们希望的是消息1如果没有被C1正确处理完毕，那么就发送给其他消费者处理，为了达到这个目的，只需要做两件事情，第一关闭rabbitMq的自动应答机制，第二消费者正确处理完消息后手动应答。

```
//1. 第二个参数autoAck设置成false表示关闭自动应答
channel.basicConsume(QUEUE_NAME, false, consumer);  
while (true) {  
    QueueingConsumer.Delivery delivery = consumer.nextDelivery();
    String message = new String(delivery.getBody());
    System.out.println(hashCode + " [x] Received '" + message + "'");
    doWork(message);
    System.out.println(hashCode + " [x] Done");
    //2. 手动应答，第二个参数multiple表示是否批量应答，很明显现在不是批量应答
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false );
}

```

## 消费者负载均衡
默认情况下rabitMq会把队列里面的消息立即发送到消费者，无论该消费者有多少消息没有应答，也就是说即使发现消费者来不及处理，新的消费者加入进来也没有办法处理已经堆积的消息，因为那些消息已经被发送给老消费者了。 

设置如下：
```
chanel.basicQos(prefetchCount) 
```
prefetchCount会告诉RabbitMQ不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack。 
这样做的好处是，如果系统处于高峰期，消费者来不及处理，消息会堆积在队列中，新启动的消费者可以马上从队列中取到消息开始工作。

![image](https://note.youdao.com/yws/api/personal/file/2C99813E6F8D47E991FD48F79634E1B4?method=download&shareKey=a78ca514993bb2c6b2013b152d180291)

上图描述了这样一个过程：刚启动时消费者C1、C2、C3均为空闲状态，且它们的channel都设置了prefetch=1，队列中有消息1~N。下面按照各消费者收到消息的时间顺序说明整个过程。

C1收到消息1开始处理。 
C2收到消息2开始处理。 
C3收到消息3开始处理。 
C2处理完消息2并产生应答，可以看到图中消息2从队列中移除了。 
C2收到消息4开始处理，可以看到图中消息4被标记为已发送状态。 
C3处理完消息3并产生应答，可以看到图中消息3从队列中移除了。 
C3收到消息5开始处理，可以看到消息5被标记为已发送状态。 
上图反应的就是直到C3收到消息5并处理时的整个队列中所有消息的状态。