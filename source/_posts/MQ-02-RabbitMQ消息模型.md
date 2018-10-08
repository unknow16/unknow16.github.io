---
title: MQ-02-RabbitMQ消息模型
date: 2018-10-08 19:03:05
tags: MQ
---

RabbitMQ 是一个在 AMQP 基础上完整的，可复用的企业消息系统。

## 交换器（Exchanges）
RabbitMQ 消息模型的核心思想是，生产者不直接发送任何消息给队列。实际上，一般的情况下，生产者甚至不知道消息应该发送到哪些队列。相反的，生产者只能将信息发送到交换器。

交换器一边收到来自生产者的消息，另一边将它们推送到队列。交换器必须准确知道接收到的消息如何处理。是否被添加到一个特定的队列吗？是否应该追加到多个队列？或者是否应该被丢弃？这些规则通过交换器类型进行定义。

交换器一共四种类型：direct、topic、headers、fanout。


## 绑定（Bindings）
创建完交换器和队列后，我们需要告诉交换器发送消息到我们的队列。交换器和队列之间的关系称为绑定。绑定是建立交换器和队列之间的关系。这可以简单地理解：队列对该交换器上的消息感兴趣。

## 1. 不用交换机的队列
![image](https://note.youdao.com/yws/api/personal/file/E9DB0587450A4D3FAE4676ACB18E63BB?method=download&shareKey=88ba6077bce11cf82df3b242963c79f7)

#### 生产者示例
```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class NoExchangeSender {
	
	private static final String QUEUE_NAME = "hello";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明队列
		channel.queueDeclare(QUEUE_NAME, false, false, false, null);
		
		// 4. 发送消息
		for(int i=0; i<10; i++) {
			String message = "Hello " + i;
			channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
			System.out.println(" [x] Sent '" + message + "'");  
		}
		 
        // 5. 关闭频道和连接  
        channel.close();
        connection.close();
	}
}

```
#### 消费者示例

```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class NoExchangeReceiver {

	private static final String QUEUE_NAME = "hello";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明队列
		channel.queueDeclare(QUEUE_NAME, false, false, false, null);
		
		// 4. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[Receiver] "  + new String(body));
        }
	}
}

```

#### API介绍
-  channel.queueDeclare(queue, durable, exclusive, autoDelete, arguments)

可以看到生产者和消费者用同样的参数声明了队列，官方推荐该做法，事实上对于一个已经存在的队列即使该方法试图用不同的参数去创建队列也不会有任何效果，这意味着不会改变队列更不会影响队列现在的工作。 

1. queue:队列名字 
1. durable:队列持久化标志,true为持久化队列 
1. exclusive：exclusive：排他队列，如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。这里需要注意三点：其一，排他队列是基于连接可见的，同一连接的不同信道是可以同时访问同一个连接创建的排他队列的。其二，“首次”，如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同。其三，即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除的。这种队列适用于只限于一个客户端发送读取消息的应用场景。  
1. autoDelete：自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。 
1. arguments：Map类型，关于队列及队列中消息的详细设置

arguments键 | 值 | 意义
--|--|--
x-message-ttl | 数字类型，标志时间，以豪秒为单位 | 标志队列中的消息存活时间，也就是说队列中的消息超过了制定时间会被删除
x-expires | 数字类型，标志时间，以豪秒为单位 | 队列自身的空闲存活时间，当前的queue在指定的时间内，没有consumer、basic.get也就是未被访问，就会被删除。
x-max-length和x-max-length-bytes|数字|最大长度和最大占用空间，设置了最大长度的队列，在超过了最大长度后进行插入会删除之前插入的消息为本次的留出空间,相应的最大占用大小也是这个道理，当超过了这个大小的时候，会删除之前插入的消息为本次的留出空间。
x-dead-letter-exchange和x-dead-letter-routing-key| 字符串|消息因为超时或超过限制在队列里消失，这样我们就丢失了一些消息，也许里面就有一些是我们做需要获知的。而rabbitmq的死信功能则为我们带来了解决方案。设置了dead letter exchange与dead letter routingkey（要么都设定，要么都不设定）那些因为超时或超出限制而被删除的消息会被推动到我们设置的exchange中，再根据routingkey推到queue中
x-max-priority|数字|队列所支持的优先级别，列如设置为5，表示队列支持0到5六个优先级别，5最高，0最低，当然这需要生产者在发送消息时指定消息的优先级别，消息按照优先级别从高到低的顺序分发给消费者

- channel.basicPublish(exchange, routingKey, mandatory, immediate, basicProperties, body)

1. exchange:  交换机名 
1. routingKey:  路由键 
1. mandatory:当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返还给生产者, channel.addReturnListener添加一个监听器，当broker执行basic.return方法时，会回调handleReturn方法，这样我们就可以处理变为死信的消息了；当mandatory设为false时，出现上述情形broker会直接将消息扔掉;通俗的讲，mandatory标志告诉broker代理服务器至少将消息route到一个队列中，否则就将消息return给发送者。 
1. immediate: rabbitMq3.0已经不在支持了，3.0以前这个标志告诉服务器如果该消息关联的queue上有消费者，则马上将消息投递给它，如果所有queue都没有消费者，直接把消息返还给生产者，不用将消息入队列等待消费者了。 
1. basicProperties：消息的详细属性，例如优先级别、持久化、到期时间，headers类型的exchange要用到的是其中的headers字段。
1. body:消息实体，字节数组。
```
public BasicProperties(
    String contentType,//消息类型如：text/plain
    String contentEncoding,//编码
    Map<String,Object> headers,
    Integer deliveryMode,//1:nonpersistent 2:persistent
    Integer priority,//优先级
    String correlationId,
    String replyTo,//反馈队列
    String expiration,//expiration到期时间
    String messageId,
    Date timestamp,
    String type,
    String userId,
    String appId,
    String clusterId)

```

- QueueingConsumer

这是一个已经实现好了的Consumer,相比于自己实现Consumer接口，这是个比较安全快捷的方式。该类基于jdk的BlockingQueue实现，handleDelivery方法中将收到的消息封装成Delivery对象，并存放到BlockingQueue中，这相当于消费者本地存放了一个消息缓存队列。nextDelivery()方法底层调用的BlockingQueue的阻塞方法take()。 

- channel.basicConsume(queue, autoAck, consumer)
1. queue:队列名。 
1. autoAck:自动应答标志，true为自动应答。 
1. consumer:消费者对象，可以自己实现Consumer接口，建议使用QueueingConsumer。

## 2. fanout类型交换器
![image](https://note.youdao.com/yws/api/personal/file/8BEF2F209E44438A86AAC07E8958264A?method=download&shareKey=8e2232794191781600297905841d1d50)

1. 消息与队列匹配规则：fanout类型交换机会将接收到的消息广播给所有与之绑定的队列。 
2. 现在我们来演示一下如图所示的消息广播机制，不难注意到这种情况生产者P只关心消息发送给哪个交换机，由交换机X决定消息发送到哪些队列，，而消费者C只关注订阅哪个队列。

#### 生产者示例

```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class FanoutSender {
	
	private static final String EXCHANGE_NAME = "hello";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
		
		// 4. 发送消息
		for(int i=0; i<10; i++) {
			String message = "Hello " + i;
			channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
			System.out.println(" [x] Sent '" + message + "'");  
		}
		 
        // 5. 关闭频道和连接  
        channel.close();
        connection.close();
	}
}

```
#### 消费者示例

```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class FanoutReceiver {

	private static final String EXCHANGE_NAME = "hello";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		// 5. 将队列绑定到交换器上
		channel.queueBind(queue, EXCHANGE_NAME, "");
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[Receiver] "  + new String(body));
        }
	}
}

```

#### API介绍
- channel.exchangeDeclare(exchange, type );  
1. exchange: 交换机名。 
1. type:交换机类型，值为fanout、direct、topic、headers其中之一。 

- channel.queueDeclare()

创建一个非持久的、唯一的、自动删除的队列且队列名称由服务器随机产生。 

- channel.queueBind(queue, exchange, bindingKey);  
1. queue:队列名。 
1. exchange:交换机名。 
1. bindingKey:绑定键

## 3. direct类型交换器
![image](https://note.youdao.com/yws/api/personal/file/592887940960434099FDA6D80C83AAB4?method=download&shareKey=7d7d6b88cfca29aefdec258d16e52465)

1. 消息分发规则：消息会被推送至绑定键（binding key）和消息发布附带的选择键（routing key）完全匹配的队列。 
2. 图示说明：消息1附带路由键“error”、与绑定键“error”匹配，而队列Q4、Q5与交换机X间都存在绑定键“error”所以消息1被分发到Q4、Q5；消息2附带路由键“info”,而队列Q4与交换机间存在绑定建“info”,所以消息2被分发到队列Q4。

3. 分发到队列的消息不再带有绑定键，事实上分发到队列的消息不再带有发送者的任何信息，当然如果消息实体里面包含了发送者消息，那么消费者可以获取发送者信息

#### 生产者示例
```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class DirectSender {
	
	private static final String EXCHANGE_NAME = "hello-direct";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "direct");
		
		// 4. 发送消息
		channel.basicPublish(EXCHANGE_NAME, "error", null, "hello-error".getBytes());
		channel.basicPublish(EXCHANGE_NAME, "info", null, "hello-info".getBytes());
		 
        // 5. 关闭频道和连接  
        channel.close();
        connection.close();
	}
}

```


#### 消费者示例
```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class DirectReceiver1 {

	private static final String EXCHANGE_NAME = "hello-direct";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "direct");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		// 5. 将队列绑定到交换器上
		channel.queueBind(queue, EXCHANGE_NAME, "error");
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[Error Receiver] "  + new String(body));
        }
	}
}


package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class DirectReceiver2 {

	private static final String EXCHANGE_NAME = "hello-direct";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "direct");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		// 5. 将队列绑定到交换器上
		channel.queueBind(queue, EXCHANGE_NAME, "error");
		channel.queueBind(queue, EXCHANGE_NAME, "info");
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[Error/Info Receiver] "  + new String(body));
        }
	}
}

```

## 4. topic类型的交换机
![image](https://note.youdao.com/yws/api/personal/file/42F0485DD45A4E22B3ED36DBF2DCC5C1?method=download&shareKey=dc4fc3cee89b4a2809b705ac5c8a681e)

1. 消息分发规则：一个附带特殊的选择键将会被转发到绑定键与之匹配的队列中。

2. routingKey是声明exchange时指定的，bindingKey是queueBind时指定的。
3. routingKey于bindingKey匹配规则：routingKey必须是由点隔开的一系列的标识符组成。标识符可以是任何东西，但是一般都与消息的某些特性相关。一些合法的选择键的例子：”stock.usd.nyse”, “nyse.vmw”,”quick.orange.rabbit”。你可以定义任何数量的标识符，上限为255个字节。绑定键和选择键的形式一样。需要注意的是：关于绑定键有两种特殊的情况。

- *可以匹配一个标识符。
- #可以匹配0个或多个标识符。


3. 图示说明：消息1附带路由键“fast.orange.*”与绑定键“#”、“*.orange.*”匹配，所以消息1被分发给队列Q6、Q7；消息2附带路由键“lazy.orange.a.b”与绑定键“#”、“lazy.#”匹配，所以消息2被分发给队列Q6、Q8。

#### 生产者示例

```
package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class TopicSender {
	
	private static final String EXCHANGE_NAME = "hello-topic";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "topic");
		
		// 4. 发送消息
		channel.basicPublish(EXCHANGE_NAME, "fast.orange.*", null, "hello-fast.orange.*".getBytes());
		channel.basicPublish(EXCHANGE_NAME, "lazy.orange.a.b", null, "hello-lazy.orange.a.b".getBytes());
		 
        // 5. 关闭频道和连接  
        channel.close();
        connection.close();
	}
}

```


#### 消费者示例

```

package com.fuyi.rabbitmq.exchange;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class TopicReceiver1 {

	private static final String EXCHANGE_NAME = "hello-topic";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "topic");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		// 5. 将队列绑定到交换器上
		channel.queueBind(queue, EXCHANGE_NAME, "#");
		//channel.queueBind(queue, EXCHANGE_NAME, "lazy.#");
		//channel.queueBind(queue, EXCHANGE_NAME, "*.orange.*");
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[# Receiver] "  + new String(body));
        }
	}
}
```


## 5. headers类型的交换机
![image](https://note.youdao.com/yws/api/personal/file/FE01297581A148498263AC78950060EA?method=download&shareKey=8857e8cd19d85581cdcc8c6fbfb060c4)

1. 消息分发规则：headers类型的交换机分发消息不依赖routingKey,是使用发送消息时basicProperties对象中的headers来匹配的。headers是一个键值对类型，发送者发送消息时将这些键值对放到basicProperties对象中的headers字段中，队列绑定交换机时绑定一些键值对，当两者匹配时，队列就可以收到消息。匹配模式有两种，在队列绑定到交换机时用x-match来指定，all代表定义的多个键值对都要满足，而any则代码只要满足一个就可以了。fanout，direct，topic exchange的routingKey都需要要字符串形式的，而headers exchange则没有这个要求，因为键值对的值可以是任何类型。

2. 图示说明： 消息1附带的键值对与Q9绑定键值对的color匹配、speed不匹配，但是Q9的x-match设置为any，因此只要有一项匹配消息1就可以被分发到Q9。消息1与Q10完全匹配，消息2与Q10部分匹配，由于Q10的x-match设置为all,所以只能接受到消息1。

#### 生产者示例

```
package com.fuyi.rabbitmq.exchange;

import java.util.HashMap;
import java.util.Map;

import com.rabbitmq.client.AMQP.BasicProperties.Builder;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class HeadersSender {
	
	private static final String EXCHANGE_NAME = "hello-headers";
	
	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "headers");
		
		
		// 4. 发送消息
		Map<String, Object> headers = new HashMap<>();
		headers.put("color", "red");
		headers.put("speed", "fast");
		Builder builder = new Builder();
		builder.headers(headers);
		channel.basicPublish(EXCHANGE_NAME, "", builder.build(), "hello-fast".getBytes());
		
		Map<String, Object> headers1 = new HashMap<>();
		headers1.put("color", "red");
		headers1.put("speed", "normal");
		Builder builder1 = new Builder();
		builder1.headers(headers1);
		channel.basicPublish(EXCHANGE_NAME, "", builder1.build(), "hello-normal".getBytes());
		 
        // 5. 关闭频道和连接  
        channel.close();
        connection.close();
	}
}

```

#### 消费者示例
```
package com.fuyi.rabbitmq.exchange;

import java.util.HashMap;
import java.util.Map;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class HeadersReceiver1 {

	private static final String EXCHANGE_NAME = "hello-headers";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "headers");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		
		
		// 5. 将队列绑定到交换器上
		Map<String, Object> headers = new HashMap<>();
		headers.put("x-match", "any");
		headers.put("color", "red");
		headers.put("speed", "low");
		channel.queueBind(queue, EXCHANGE_NAME, "", headers);
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[x-match any Receiver] "  + new String(body));
        }
	}
}


package com.fuyi.rabbitmq.exchange;

import java.util.HashMap;
import java.util.Map;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.QueueingConsumer.Delivery;

public class HeadersReceiver2 {

	private static final String EXCHANGE_NAME = "hello-headers";

	public static void main(String[] args) throws Exception {
		// 1. 创建连接
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost("192.168.0.221");
		connectionFactory.setUsername("hzx_admin");
		connectionFactory.setPassword("123456");
		Connection connection = connectionFactory.newConnection();
		
		// 2. 通过连接创建通道
		Channel channel = connection.createChannel();
		
		// 3. 声明交换器
		channel.exchangeDeclare(EXCHANGE_NAME, "headers");
		// 4. 创建一个非持久的、唯一的且自动删除的队列  
		String queue = channel.queueDeclare().getQueue();
		
		
		// 5. 将队列绑定到交换器上
		Map<String, Object> headers = new HashMap<>();
		headers.put("x-match", "all");
		headers.put("color", "red");
		headers.put("speed", "fast");
		channel.queueBind(queue, EXCHANGE_NAME, "", headers);
		
		// 6. 创建队列消费者
		QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queue, true, consumer);
        
        // 5. 获取数据
        for(;;) {
        	Delivery nextDelivery = consumer.nextDelivery();
        	byte[] body = nextDelivery.getBody();
        	System.out.println("[x-match all Receiver] "  + new String(body));
        }
	}
}

```

#### API介绍
-  channel.exchangeDeclare(exchange, type, durable, autoDelete, arguments)；
1. exchange:交换机名 
1. type：交换机类型 
1. durable:持久化标志 
1. autoDelete:自动删除 
1. arguments:扩展参数，具体如下表

扩展参数键|意义
--|--
alternate-exchange|下面简称AE，当一个消息不能被route的时候，如果exchange设定了AE，则消息会被投递到AE。如果存在AE链，则会按此继续投递，直到消息被route或AE链结束或遇到已经尝试route过消息的AE。

- channel.queueBind(queue, exchange, routingKey, arguments)； 
1. queue:队列名 
1. exchange:交换机名 
1. routingKey:选择键（路由键） 
1. arguments:headers类型交换机绑定时指定的键值对