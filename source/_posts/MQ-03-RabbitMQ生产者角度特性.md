---
title: MQ-03-RabbitMQ生产者角度特性
date: 2018-10-08 19:04:20
tags: MQ
---

在交换机简介一篇中，通过生产者和消费者都声明同样的交换机同样的队列，可以保证无论生产者还是消费者先运行起来，都不会由于交换机或者队列不存在而出现错误。但是上一篇所有的代码都存在如下两个问题： 
1. 服务器宕机，消息将丢失。 
2. 发送的消息如果没有正确路由到队列，例如生产者先运行消费者未运行的情况，那么消息将被丢弃，且生产者没有收到任何反馈。 

下面的介绍将会解决这些问题。

## 消息持久化
以前我的消息都是临时消息，服务器宕机重启后，没有被处理的消息将会被丢弃掉，通过设置交换机持久化、队列持久化、消息持久化可以解决这一个问题。

#### 生产者示例

```
package com.fuyi.rabbitmq.durable;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class Sender {

	private static final String EXCHANGE_NAME = "hello-direct-durable";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明持久化交换机
		boolean durable = true;
		channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable);
		
		// 4. 指定发送消息为持久化， MessageProperties类定义了一系列常用BasicProperties的集合
		for(int i=0; i<100000; i++) {
			channel.basicPublish(EXCHANGE_NAME, "", MessageProperties.PERSISTENT_TEXT_PLAIN, ("hello-direct-durable-" + i ).getBytes());
		}
		
		// 5. 关闭连接
		channel.close();
		connection.close();
	}
}


```

#### 消费者示例

```
package com.fuyi.rabbitmq.durable;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class Receiver {

	private static final String EXCHANGE_NAME = "hello-direct-durable";
	private static final String QUEUE_NAME = "hello-direct-durable";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明持久化交换机
		boolean durable = true;
		channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable);
		
		// 4. 声明持久化队列
		channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
		
		// 5. 将队列绑定到交换机上
		channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
		
		// 6. 从队列获取消息
		QueueingConsumer consumer = new QueueingConsumer(channel);
		channel.basicConsume(QUEUE_NAME, true, consumer);
		for(;;) {
			Delivery delivery = consumer.nextDelivery();
			// 获取路由键
			// String routingKey = delivery.getEnvelope().getRoutingKey();
			byte[] body = delivery.getBody();
			System.out.println("[Receiver] : " + new String(body));
		}
		
	}
}

```
即使交换机和队列都是持久化的，消息是否持久化取决于发送方发送消息时指定的properties，MessageProperties类定义了一系列常用BasicProperties的集合，例如持久化消息MessageProperties.PERSISTENT_TEXT_PLAIN。

## 事务和消息确认

消息持久化，可以保证已经发送到队列中的消息被持久化到磁盘上，服务器重启后可以从磁盘恢复这些消息。那么对于正在发送还没有成功的消息呢？两种解决方案：

#### 1. 事务

RabbitMQ中与事务机制有关的方法有三个，分别是Channel里面的txSelect()，txCommit()以及txRollback()，txSelect用于将当前Channel设置成是transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定是到达broker了，如果在txCommit执行之前broker异常奔溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

```
channel.txSelect();
try{
    channel.basicPublish(EXCHANGE_NAME, "blue", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes()); 
    System.out.println("commit");
    channel.txCommit();
}catch(Exception e){
    System.out.println("rollback");
    channel.txRollback();
}

```

#### 2. PublisherConfirm

Publisher Confirm机制（又称为Confirms或Publisher Acknowledgements）是作为解决事务机制性能开销大（导致吞吐量下降）而提出的另外一种保证消息不会丢失的方式。

生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

- 优点

confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息。

在channel 被设置成 confirm 模式之后，所有被 publish 的后续消息都将被 confirm（即 ack） 或者被nack一次。但是没有对消息被 confirm 的快慢做任何保证，并且同一条消息不会既被 confirm又被nack 。

- 缺点

Confirm机制在性能上要比事务优越很多。但是Confirm机制，无法进行回滚，就是一旦服务器崩溃，生产者无法得到Confirm信息，生产者其实本身也不知道该消息吃否已经被持久化，只有继续重发来保证消息不丢失，但是如果原先已经持久化的消息，并不会被回滚，这样队列中就会存在两条相同的消息，系统需要支持去重。

#### 生产者示例
```
package com.fuyi.rabbitmq.ack;

import java.io.IOException;
import java.util.Collections;
import java.util.SortedSet;
import java.util.TreeSet;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConfirmListener;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class ConfirmSender {

	private static final String EXCHANGE_NAME = "hello-direct-durable";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明持久化交换机
		boolean durable = true;
		channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable);
		
		// 4. 发消息，确认机制通过confirm实现
		final SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());
		channel.addConfirmListener(new ConfirmListener() {
			
			@Override
			public void handleNack(long deliveryTag, boolean multiple) throws IOException {
				System.out.println("noack, deliveryTag: " + deliveryTag + ", multiple: " + multiple);
			}
			
			@Override
			public void handleAck(long deliveryTag, boolean multiple) throws IOException {
				System.out.println("ack, deliveryTag: " + deliveryTag + ", multiple: " + multiple);
				if(multiple) {
					confirmSet.headSet(deliveryTag + 1).clear();
				} else {
					confirmSet.remove(deliveryTag);
				}
			}
		});
		
		long nextPublishSeqNo = channel.getNextPublishSeqNo();
		channel.basicPublish(EXCHANGE_NAME, "", MessageProperties.PERSISTENT_TEXT_PLAIN, ("hello-direct-durable-confirm").getBytes());
		confirmSet.add(nextPublishSeqNo);
		
		// 5. 关闭连接
		channel.close();
		connection.close();
	}
}

```
## mandatory

事务机制和消息确认机制都是为了保证异常状态下的消息不丢失，其实正常状态下也可能存在消息丢失问题，例如交换机按照路由规则未找到该消息对应的队列。confirm机制配合mandatory标志使用可以实现消息发送的可靠性，且性能较好。 

当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返还给生产者, channel.addReturnListener添加一个监听器，当broker执行basic.return方法时，会回调handleReturn方法，这样我们就可以处理变为死信的消息了；当mandatory设为false时，出现上述情形broker会直接将消息扔掉;通俗的讲，mandatory标志告诉broker代理服务器至少将消息route到一个队列中，否则就将消息return给发送者。

#### 生产者示例
```
package com.fuyi.rabbitmq.mandatory;

import java.io.IOException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;
import com.rabbitmq.client.ReturnListener;

public class Sender {

	private static final String EXCHANGE_NAME = "hello-direct-durable";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明持久化交换机
		boolean durable = true;
		channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable);
		
		// 当mandatory=true时，添加该监听器回调处理死信消息
		channel.addReturnListener(new ReturnListener() {
			
			@Override
			public void handleReturn(int replyCode,String replyText,String exchange,
					String routingKey,AMQP.BasicProperties properties,byte[] body) throws IOException {
				System.out.println("handleReturn");
				System.out.println("replyCode:"+replyCode);
				System.out.println("replyText:"+replyText);
				System.out.println("messagebody:"+new String(body));
			}
			
		});

		// 4. 发送消息
		boolean mandatory = true;
		channel.basicPublish(EXCHANGE_NAME, "hehe", mandatory, MessageProperties.PERSISTENT_TEXT_PLAIN, ("hello-direct-durable-mandatory").getBytes());
		
		
		System.in.read();
		
		// 5. 关闭连接
		channel.close();
		connection.close();
	}
}

```
