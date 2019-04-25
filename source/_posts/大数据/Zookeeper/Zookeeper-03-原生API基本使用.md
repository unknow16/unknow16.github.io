---
title: Zookeeper-03-原生API基本使用
date: 2018-06-13 15:23:55
tags: Zookeeper
---

## Watcher机制
#### pom.xml

```
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>3.4.6</version>
</dependency>
```

#### 1. 实现Watcher接口
```
package com.fuyi.zk.watcher;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;

public class MyWatcher implements Watcher {

	public void process(WatchedEvent event) {
		System.out.println(event);
	}

}

```
#### 2. 使用ZooKeeper构造中的默认watcher
```
@Test
public void test1() throws Exception {
	//0. 创建watcher
	MyWatcher watcher = new MyWatcher();
	
	//1. 在创建ZooKeeper时第三个参数负责设置该zk连接的默认watcher
	//   在建立ZooKeeper连接时，就会触发watcher
	ZooKeeper zk = new ZooKeeper("localhost:2181", 3000, watcher);
	
	//2. 设置watcher：第二个参数设置true，表示用1步中默认watcher
	//   此处可传入watcher实例
	Stat stat = zk.exists("/testWatcher", true);
	
	//3. create时 触发
	zk.create("/testWatcher", "watcher".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
}

output:
WatchedEvent state:SyncConnected type:None path:null // 创建时输出
WatchedEvent state:SyncConnected type:NodeCreated path:/testWatcher // create时输出
```
---
```
@Test
public void test2() throws Exception {
	//0. 创建watcher
	MyWatcher watcher = new MyWatcher();
	
	//1. 在创建ZooKeeper时第三个参数负责设置该zk连接的默认watcher
	//   在建立ZooKeeper连接时，就会触发watcher
	ZooKeeper zk = new ZooKeeper("localhost:2181", 3000, watcher);
	
	//2. 创建node
	zk.create("/testWatcher", "watcher".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
	
	//3. 设置两个监听，只会收到一个通知
	Stat stat = zk.exists("/testWatcher", true);
	zk.getData("/testWatcher", true, stat);
	
	//4. 触发通知 
	zk.setData("/testWatcher", "fuyi".getBytes(), stat.getVersion());
}

output:
WatchedEvent state:SyncConnected type:None path:null // zk创建时输出
WatchedEvent state:SyncConnected type:NodeDataChanged path:/testWatcher //setData()时触发 
```

---

```
@Test
public void test3() throws Exception {
	//0. 创建watcher
	MyWatcher watcher = new MyWatcher();
	
	//1. 创建zk
	ZooKeeper zk = new ZooKeeper("localhost:2181", 3000, watcher);
	
	//2. 创建/root,只有持久化的节点才能有子节点,此处是持久化节点，
	//   多次运行会报节点存在错误,可注掉
	//zk.create("/root", "watcher".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
	
	//3. 设置/root节点，的孩子监听
	zk.getChildren("/root", true);
	
	//4. 触发/root节点的孩子节点改变事件
	zk.create("/root/fuyi", "watcher".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
}

output:
WatchedEvent state:SyncConnected type:None path:null
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/root
```

#### 3. 使用不同watcher实例
```
// 每次调用，获取Watcher实例
private Watcher getInstance(final String msg) {
	return new Watcher() {
		
		public void process(WatchedEvent event) {
			System.out.println(msg + ":  " + event.toString());
		}
	};
}

@Test
public void test4() throws Exception {
	// 0.创建zk
	ZooKeeper zk = new ZooKeeper("localhost:2181", 3000, null);
	
	// 1.创建节点
	zk.create("/root/fuyi1", "fff".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
	
	// 2.设置监听
	Stat stat = zk.exists("/root/fuyi1", getInstance("exists"));
	zk.getData("/root/fuyi1", getInstance("getData"), stat);
	zk.getChildren("/root", getInstance("getChildren"));
	
	// 3.触发监听
	zk.delete("/root/fuyi1", stat.getVersion());
}

output:
getData:  WatchedEvent state:SyncConnected type:NodeDeleted path:/root/fuyi1
exists:  WatchedEvent state:SyncConnected type:NodeDeleted path:/root/fuyi1
getChildren:  WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/root

```
上例中由于exists和getData分别设置了两个不同的Watcher实例，所以虽然watcher都是由同了一个NodeDataChanged触发的，但exists()和getData()都会收到通知。由于创建Zookeeper时没有设置watcher(参数为null)，所以建立连接时没有收到通知。