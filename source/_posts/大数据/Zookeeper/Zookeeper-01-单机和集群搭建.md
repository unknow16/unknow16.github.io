---
title: Zookeeper-01-单机和集群搭建
date: 2018-06-13 15:23:24
tags: Zookeeper
---

## 单机搭建
1. 下载zookeeper的安装包之后, 解压到合适目录
2. 进入zookeeper目录下的conf子目录，复制zoo_sample.cfg为zoo.cfg
3. 启动服务，命令如下：
```
// 启动命令
bin/zkServer.sh start

// 其他命令
// 查看状态
bin/zkServer.sh status
// 关闭
bin/zkServer.sh stop
// 命令行客户端连接服务器
bin/zkCli.sh -server localhost:2181
// 查看JVM进程名
jps //显示Java进程名为QuorumPeerMain
```

## 集群搭建
1. 下载zookeeper的安装包之后, 解压到合适目录
2. 进入zookeeper目录下的conf子目录，复制zoo_sample.cfg为zoo.cfg
3. 更改配置（先在一台节点上配置）

```
dataDir=/home/hadoop/app/zookeeper-3.4.5/data
server.1=yarn01:2888:3888
server.2=yarn02:2888:3888
server.3=yarn03:2888:3888
```

4. 在dataDir下创建一个myid文件，里面内容是server.N中的N
    
```
// 可用该命令实现
echo 1 > myid
```

5. 将配置好的zk拷贝到其他节点

```
scp -r zookeeper-3.4.5/ yarn02:~/app/
scp -r zookeeper-3.4.5/ yarn03:~/app/
```
6. 分别启动服务即可，启动命令如上，可查看相应状态，一个为Leader，另外两个为flower

注意：
* 在其他节点上一定要修改myid的内容
* yarn01,yarn02,yarn03为域名，也可指定为ip

## 配置文件解析
```
tickTime=2000 # 每个tick的毫秒数
initLimit=5
syncLimit=2

dataDir=/Users/apple/zookeeper0/data # 数据存放目录  
dataLogDir=/Users/apple/zookeeper0/logs  # 日志存放目录
clientPort=2181 # 监听客户端访问的端口

#maxClientCnxns=60 # 客户端连接的最大数
#autopurge.snapRetainCount=3 # 保留在dataDir中的快照数量
#autopurge.purgeInterval=1 # 每小时自动执行一次清除操作，设置0失效自动清除
```
#### tickTime：CS通信心跳时间
Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。

tickTime=2000:表示2秒发一次心跳检测

#### initLimit：LF初始通信时限
zookeeper集群中的包含多台server, 其中一台为leader, 集群中其余的server为follower. 

集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量）。

此时该参数设置为5, 说明时间限制为5倍tickTime, 即5*2000=10000ms=10s.

#### syncLimit：LF同步通信时限
集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）。

此时该参数设置为2, 说明时间限制为2倍tickTime, 即4000ms.

#### dataDir：数据文件目录，必须设置
Zookeeper保存数据的目录，默认情况下，Zookeeper将写数据的日志文件也保存在这个目录里。

#### dataLogDir: 日志文件目录
如果没有设置该参数, 将使用和dataDir相同的设置.

#### clientPort：客户端连接端口
客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

一般端口为2181。

#### 集群信息
服务器名称与地址：

```
// 代表含义依次为：
// 服务器编号，服务器地址，LF通信端口，选举端口
server.N=YYY:A:B
```

