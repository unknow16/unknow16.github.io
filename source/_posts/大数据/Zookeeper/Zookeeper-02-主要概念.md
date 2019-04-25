---
title: Zookeeper-02-主要概念
date: 2018-06-13 15:23:44
tags: Zookeeper
---

zookeeper在读写比例为10:1时性能最佳。

每个znode上data的读写都是原子操作。

读是局部性的，即client只需要从与它相连的server上读取数据即可；而client有写请求的话，与之相连的server会通知leader，然后leader会把写操作分发给所有server。所以定要比读慢很多。

ACL不是递归的，它只针对当前节点，对子节点没有任何影响。

### CreateMode

- PERSISTENT：创建后只要不删就永久存在
- EPHEMERAL：会话结束年结点自动被删除，EPHEMERAL结点不允许有子节点
- PERSISTENT_SEQUENTIAL：节点名末尾会自动追加一个10位数的单调递增的序号，同一个节点的所有子节点序号是单调递增的,且是持久化的
- EPHEMERAL_SEQUENTIAL：与前者区别为非持久化，会话结束即自动删除

### Watch机制
Znode发生变化（Znode本身的增加，删除，修改，以及子Znode的变化）可以通过Watch机制通知到客户端。那么要实现Watch，就必须实现org.apache.zookeeper.Watcher接口，并且将实现类的对象传入到可以Watch的方法中。Zookeeper中所有读操作（getData()，getChildren()，exists()）都可以设置Watch选项。Watch事件具有one-time trigger（一次性触发）的特性，如果Watch监视的Znode有变化，那么就会通知设置该Watch的客户端。

在上述说道的所有读操作中，如果需要Watcher，我们可以自定义Watcher，如果是Boolean型变量，当为true时，则使用系统默认的Watcher，系统默认的Watcher是在Zookeeper的构造函数中定义的Watcher。参数中Watcher为空或者false，表示不启用Wather。

在服务端用一个Map存放watcher,所以相同的watcher只会出现一次，只要watcher被调用一次，就会被删除，所以watcher是一次性的。如getData()和exists()设置的同一个watcher，调用setData()时触发调用，但getData()和exists()只会有一个接收到通知。

### WatchEvent
- NodeDataChanged
- NodeDeleted
- NodeChildrenChanged
- NodeCreated

### Watcher类型
Znode改变有很多种方式，例如：节点创建，节点删除，节点改变，子节点改变等等。Zookeeper维护了两个Watch列表，一个节点数据Watch列表，另一个是子节点Watch列表。getData()和exists()设置数据Watch，getChildren()设置子节点Watch。两者选其一，可以让我们根据不同的返回结果选择不同的Watch方式，getData()和exists()返回节点的内容，getChildren()返回子节点列表。因此，setData()触发内容Watch，create()触发当前节点的内容Watch或者是其父节点的子节点Watch。delete()同时触发父节点的子节点Watch和内容Watch，以及子节点的内容Watch


### 设置Watcher
* data watches: 观察当前节点数据变化，可通过getData() 和 exists() 设置观察者
* children watches: 观察当前节点的孩子节点变化，可通过getChildren() 设置观察者

### 触发Watcher
* setData() 会触发 当前节点data watcher， 即NodeDataChanged
* create() 会触发data watches和child watches，即NodeDataChanged和NodeChildrenChanged
* delete() 会触发data watches和child watches，即三种事件都会触发


