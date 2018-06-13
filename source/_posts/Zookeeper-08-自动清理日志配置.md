---
title: Zookeeper-08-自动清理日志配置
date: 2018-06-13 15:24:56
tags: Zookeeper
---

在使用zookeeper过程中，我们知道，会有dataDir和dataLogDir两个目录，分别用于snapshot和事务日志的输出（默认情况下只有dataDir目录，snapshot和事务日志都保存在这个目录中，正常运行过程中，ZK会不断地把快照数据和事务日志输出到这两个目录，并且如果没有人为操作的话，ZK自己是不会清理这些文件的，需要管理员来清理，这里介绍4种清理日志的方法。

1. 写一个删除日志脚本，每天定时执行即可


2. 使用ZK的工具类PurgeTxnLog，它的实现了一种简单的历史文件清理策略，简单使用如下：

```
Java -cp zookeeper.jar:lib/slf4j-api-1.6.1.jar:lib/slf4j-log4j12-1.6.1.jar:lib/log4j-1.2.15.jar:conf org.apache.zookeeper.server.PurgeTxnLog <dataDir><snapDir> -n <count>
```

3. 对于上面这个Java类的执行，ZK自己已经写好了脚本，在bin/zkCleanup.sh中，所以直接使用这个脚本也是可以执行清理工作的。

4. 从3.4.0开始，zookeeper提供了自动清理snapshot和事务日志的功能，通过配置 autopurge.snapRetainCount 和 autopurge.purgeInterval 这两个参数能够实现定时清理了。这两个参数都是在zoo.cfg中配置的。

* autopurge.purgeInterval  

这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能。

* autopurge.snapRetainCount 
 
这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。


```
// 这样是指：48小时清理一次，清理时保留最新20个文件
// 即保留48小时内的日志，并且保留20个文件
autopurge.snapRetainCount=20  
autopurge.purgeInterval=48
```
