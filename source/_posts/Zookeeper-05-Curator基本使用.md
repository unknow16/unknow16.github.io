---
title: Zookeeper-05-Curator基本使用
date: 2018-06-13 15:24:18
tags: Zookeeper
---

Curator是Netflix公司开源的一个zookeeper客户端，现在是Apache的顶级项目，官网：http://curator.apache.org/

#### pom.xml

```
<!-- curator -->
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
	<version>2.4.2</version>
</dependency>
```

#### 创建连接

```
// 重试策略，初试1秒，重试10次
RetryPolicy retryRolicy = new ExponentialBackoffRetry(1000, 10);
private CuratorFramework cf;

@Before
public void test1() {
	cf = CuratorFrameworkFactory.builder()
								.connectString("localhost:2181")
								.connectionTimeoutMs(5000)
								.retryPolicy(retryRolicy)
								//.namespace("app1") //设置一个存在的节点，使用该链接的操作，都相对在该节点下创建
								.build();
	// 2.开启连接
	cf.start();
	System.out.println("连接状态：" + cf.getState());
}
```

#### 增删该查

```
@Test
public void test2() throws Exception {
	// 创建节点，不通过withMode设置模式，则默认为持久化节点
	// cf.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT).forPath("/aa", "fff".getBytes());
	
	// 删除节点
	// cf.delete().forPath("/aa");
	
	// 读取节点
	// byte[] data = cf.getData().forPath("/aa");
	// System.out.println(new String(data));
	 
	// 修改节点内容
	// cf.setData().forPath("/aa", "bbb".getBytes());
	
	// 读取节点
	// data = cf.getData().forPath("/aa");
	// System.out.println(new String(data));
	 
	 // 判读节点是否存在
	 Stat stat = cf.checkExists().forPath("/app1");
	 System.out.println(stat);
}
```

#### 绑定回调函数

```
@Test
public void test3() throws Exception {
	cf.create()
		.creatingParentsIfNeeded()
		.inBackground(new BackgroundCallback(){

			public void processResult(CuratorFramework cf, CuratorEvent event) throws Exception {
				System.out.println(cf);
				System.out.println(event);
				System.out.println("path: " + event.getPath());
				System.out.println("type: " + event.getType());
				
				
			}}, Executors.newCachedThreadPool())
		.forPath("/bb", "bb".getBytes());
	
	TimeUnit.SECONDS.sleep(3);
}
```

#### 读取子节点

```
@Test
public void test4() throws Exception {
	List<String> children = cf.getChildren().forPath("/app1");
	for (String s : children) {
		System.out.println(s);
	}
}
```
