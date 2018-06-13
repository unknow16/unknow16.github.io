---
title: Zookeeper-04-ZkClient基本使用
date: 2018-06-13 15:24:08
tags: Zookeeper
---

## ZkClient简介
直接使用zk的api实现业务功能比较繁琐。因为要处理session loss，session expire等异常，在发生这些异常后进行重连。又因为ZK的watcher是一次性的，如果要基于wather实现发布/订阅模式，还要自己包装一下，将一次性订阅包装成持久订阅。另外如果要使用抽象级别更高的功能，比如分布式锁，leader选举等，还要自己额外做很多事情。ZkClient可以分别解决上述小问题。

zkClient主要做了两件事情。一件是在session loss和session expire时自动创建新的ZooKeeper实例进行重连。另一件是将一次性watcher包装为持久watcher。后者的具体做法是简单的在watcher回调中，重新读取数据的同时再注册相同的watcher实例。

## 创建会话
```
public ZkClient(String serverstring)

public ZkClient(String zkServers, int connectionTimeout)

public ZkClient(String zkServers, int sessionTimeout, int connectionTimeout)

public ZkClient(String zkServers, int sessionTimeout, int connectionTimeout, ZkSerializer zkSerializer)

public ZkClient(final String zkServers, final int sessionTimeout, final int connectionTimeout, final ZkSerializer zkSerializer, final long operationRetryTimeout)

public ZkClient(IZkConnection connection)

public ZkClient(IZkConnection connection, int connectionTimeout)

public ZkClient(IZkConnection zkConnection, int connectionTimeout, ZkSerializer zkSerializer)

public ZkClient(final IZkConnection zkConnection, final int connectionTimeout, final ZkSerializer zkSerializer, final long operationRetryTimeout)
```
上面方法的参数如果我们熟悉原生API的话，不难理解其参数，基本上参数名都是自描述的。值得留意的是ZkClient将ZK原生API中的异步处理进行了同步化。

## 创建节点
```
public void createPersistent(String path)

public void createPersistent(String path, boolean createParents)

public void createPersistent(String path, boolean createParents, List<ACL> acl)

public void createPersistent(String path, Object data)

public void createPersistent(String path, Object data, List<ACL> acl)

public String createPersistentSequential(String path, Object data)

public String createPersistentSequential(String path, Object data, List<ACL> acl) 

public void createEphemeral(final String path)

public void createEphemeral(final String path, final List<ACL> acl)

public String create(final String path, Object data, final CreateMode mode)

public String create(final String path, Object data, final List<ACL> acl, final CreateMode mode) 

public void createEphemeral(final String path, final Object data)

public void createEphemeral(final String path, final Object data, final List<ACL> acl)

public String createEphemeralSequential(final String path, final Object data)

public String createEphemeralSequential(final String path, final Object data, final List<ACL> acl)
```
查看源代码可知，其实这些创建节点的方法都是对原生API的一层封装而已，底层实现基本相同。值得留意的一点是，原生API的参数通过byte[]来传递节点内容，而ZkClient支持自定义序列化，因此可以传输Object对象。

createParents参数决定了是否递归创建父节点。true表示递归创建，false表示不使用递归创建。这也正是ZkClient帮开发人员省去了不少繁琐的检查和创建父节点的过程。

## 删除节点
```
public boolean delete(final String path)

public boolean delete(final String path, final int version)

public boolean deleteRecursive(String path)
```
删除API其实很简单，重点说一下deleteRecursive接口，这个接口提供了递归删除的功能。在原生API中，如果一个节点存在子节点，那么它将无法直接删除，必须一层层遍历先删除全部子节点，然后才能将目标节点删除。

## 读取节点数据
* 读取当前节点

```
public <T extends Object> T readData(String path)

public <T extends Object> T readData(String path, boolean returnNullIfPathNotExists)

public <T extends Object> T readData(String path, Stat stat)
```
通过方法返回参数的定义，就可以得知，返回的结果（节点的内容）已经被反序列化成对象了。

对本接口实现监听的接口为IZkDataListener，分别提供了处理数据变化和删除操作的监听：

```
public void handleDataChange(String dataPath, Object data) throws Exception;

public void handleDataDeleted(String dataPath) throws Exception;
```


* 读取当前节点的孩子节点
```
public List<String> getChildren(String path)
```

此接口返回子节点的相对路径列表。比如节点路径为/test/a1和/test/a2，那么当path为/test时，返回的结果为[a1,a2]。

其中在原始API中，对节点注册Watcher，当节点被删除或其下面的子节点新增或删除时，会通知客户端。在ZkClient中，通过Listener监听来实现，后续会将到具体的使用方法。

可以注册的Listener为，接口IZkChildListener下面的方法来实现：


```
public void handleChildChange(String parentPath, List<String> currentChilds)
```

## 更新节点数据
更新操作可以通过以下方法来实现：


```
public void writeData(String path, Object object)

public void writeData(final String path, Object datat, final int expectedVersion)

public Stat writeDataReturnStat(final String path, Object datat, final int expectedVersion)
```

## 监测节点是否存在
此API比较简单，调用以下方法即可：


```
protected boolean exists(final String path, final boolean watch)
```

## 对象序列化存储
ZkClient的构造方法中，有包含入参为ZkSerializer接口的构造方法。


```
// ZkSerializer接口定义
public interface ZkSerializer {

    public byte[] serialize(Object data) throws ZkMarshallingError;

    public Object deserialize(byte[] bytes) throws ZkMarshallingError;
}
```


参数ZkSerializer是一个接口，定义了byte数组序列化和反序列化的两个方法。如果不传递此参数，则默认使用org.I0Itec.zkclient.serialize.SerializableSerializer实现类进行序列化。某些情况下此序列化接口会出现问题，比如乱码。此时，开发者可以直接实现ZkSerializer接口，重写自己的序列化方法。比如使用Hessian或Kryo等。


```
@Test
public void test4() throws Exception {
	// 此处和用new ZkClient("localhost:2181")效果一样
	// 不传ZkSerializer实现，默认用SerializableSerializer
	ZkClient zkClient = new ZkClient("localhost:2181",5000, 1000, new SerializableSerializer());
	
	// User为自定义实体
	zkClient.createPersistent("/testData", new User("fuyi", "123"));
	
	User data = zkClient.readData("/testData");
	System.out.println(data);
	
	zkClient.delete("/testData");
	zkClient.close();
}

output:
User [username=fuyi, password=123]
```


## 订阅孩子节点变化

```
@Test
public void test1() throws Exception {
	ZkClient zkClient = new ZkClient("localhost:2181");
	
	zkClient.createPersistent("/root");
	zkClient.subscribeChildChanges("/root", new IZkChildListener() {
		
		public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
			System.out.println("parentPath: " + parentPath);
			for (String s : currentChilds) {
				System.out.println("childs: " + s);
			}
		}
	});
	
	// 1.创建子节点触发
	TimeUnit.MILLISECONDS.sleep(1000);
	zkClient.createPersistent("/root/fuyi", true);
	
	// 2.删除子节点触发
	TimeUnit.MILLISECONDS.sleep(1000);
	zkClient.delete("/root/fuyi");
	
	// 3.删除监听节点，即父节点触发
	TimeUnit.MILLISECONDS.sleep(1000);
	zkClient.delete("/root");
	
	TimeUnit.MILLISECONDS.sleep(5000);
}

output:
// 1.输出
parentPath: /root
childs: fuyi
// 2.输出
parentPath: /root
// 3.输出
parentPath: /root
	
```

## 订阅节点数据改变和节点删除

```
@Test
public void test2() throws Exception {
	ZkClient zkClient = new ZkClient("localhost:2181");
	
	zkClient.createPersistent("/root");
	zkClient.subscribeDataChanges("/root", new IZkDataListener() {
		
		public void handleDataDeleted(String dataPath) throws Exception {
			System.out.println(dataPath + " has deleted");
		}
		
		public void handleDataChange(String dataPath, Object data) throws Exception {
			System.out.println("Data of " + dataPath + " has changed, new data is " + data);
		}
		
	});
	
	// 1. 写数据触发
	TimeUnit.MILLISECONDS.sleep(1000);
	zkClient.writeData("/root", "bbbbbbbb", -1);
	
	// 2. 删节点触发
	TimeUnit.MILLISECONDS.sleep(1000);
	zkClient.delete("/root");
	
	TimeUnit.MILLISECONDS.sleep(5000);
}

output:
// 1.触发
Data of /root has changed, new data is bbbbbbbb
// 2.触发
/root has deleted
```

## 订阅会话状态改变
* 基本不用
```
@Test
public void test3() throws Exception {
	ZkClient zkClient = new ZkClient("localhost:2181");
	
	zkClient.subscribeStateChanges(new IZkStateListener() {
		
		public void handleStateChanged(KeeperState state) throws Exception {
			System.out.println("handleStateChanged");
		}
		
		public void handleSessionEstablishmentError(Throwable error) throws Exception {
			System.out.println("handleSessionEstablishmentError");
		}
		
		public void handleNewSession() throws Exception {
			System.out.println("handleNewSession");
		}
	});
}
```
