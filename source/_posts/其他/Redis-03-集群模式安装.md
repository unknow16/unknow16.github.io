---
title: Redis-03-集群模式安装
date: 2018-06-22 18:42:37
tags: Redis
---

## 准备环境
1. 系统环境

系统：CentOS6.9系统

IP和端口划分：要让集群正常工作至少需要3个主节点，在这里我们要创建6个redis节点，其中三个为主节点，三个为从节点，对应的redis节点的ip和端口对应关系如下：
```
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
127.0.0.1:7004
127.0.0.1:7005
```



2. 创建集群需要的目录
```
mkdir -p /usr/local/redis-3.2.9-cluster
cd /usr/local/redis-3.2.9-cluster
mkdir 7000
mkdir 7001
mkdir 7002
mkdir 7003
mkdir 7004
mkdir 7005
```
3. 将单机模式下编译好的可执行文件分别复制到上面创的目录中


4. 修改配置文件redis.conf

* 7000端口的配置文件如下改动，其他端口的配置文件作相应修改
```
daemonize yes                          #redis后台运行
pidfile   /var/run/redis_7000.pid      #pidfile文件对应7000, 7001, 7002
port 7000                              #端口7000, 7001, 7002
cluster-enabled  yes                   #开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf   #集群的配置  配置文件首次启动自动生成 7000,7001,7002，存在/home/xm6f/dev/redis-3.2.4/src目录
cluster-node-timeout  5000             #请求超时，设置5秒即可
appendonly  yes                        #aof日志开启，有需要就开启，它会每次写操作都记录一条日志
logfile "/home/xm6f/dev/redis_cluster/7000/logs/redis.log"
bind 192.168.1.105 #绑定当前服务器的IP，否则的话在集群通信的时候会出现：[ERR] Sorry, can't connect to node 192.168.200.140:7001

dbfilename dump_7000.rdb #存在/home/xm6f/dev/redis-3.2.4/src目录
appendfilename "appendonly_7000.aof" #存在/home/xm6f/dev/redis-3.2.4/src目录
requirepass 123456 #设置密码，每个节点的密码都必须一致的
masterauth 123456 # 若master配置了密码则slave也要配置相应的密码参数否则无法进行正常复制的
```

5. 为每个端口开启防火墙通行

## 启动所有redis服务

两种启动方式：
1. 进入7000、7001、7002等目录分别启动

```
$ ./redis-server redis.conf # 以指定配置文件启动
```

2. 编写启动脚本start-all.sh, 如下：

```
cd /usr/local/redis-3.2.9-cluster

cd 7000/
./redis-server redis.conf

cd ../7001
./redis-server redis.conf

cd ../7002
./redis-server redis.conf

cd ../7003
./redis-server redis.conf

cd ../7004
./redis-server redis.conf

cd ../7005
./redis-server redis.conf
```

查看服务是否启动完成
```
ps -ef | grep redis #查看是否启动成功

netstat -tnlp | grep redis #可以看到redis监听端口
```

## 创建集群
前面已经准备好了搭建集群的redis节点，接下来我们要把这些节点都串连起来搭建集群。官方提供了一个工具：redis-trib.rb(redis-3.2.9/src/redis-trib.rb)，可将其复制到/usr/local/redis-3.2.9-cluster/下， 看后缀就知道这东西不能直接执行，它是用ruby写的一个程序，所以我们还得安装ruby。
```
yum -y install ruby ruby-devel rubygems rpm-build
```
再用 gem 这个命令来安装 redis 接口，gem是ruby的一个工具包。
```
gem install redis //等一会儿就好了
```
注意：在执行gem install redis时，报ERROR:Error installing redis:redis requires Ruby version >= 2.2.2异常。解决方案[传送门](https://blog.csdn.net/fengye_yulu/article/details/77628094)

上面的步骤完事了，接下来运行一下redis-trib.rb

```
./redis-trib.rb # 会显示命令帮助
```
看到这，应该明白了吧，就是靠上面这些操作完成redis集群搭建。

确认所有的节点都启动，接下来使用参数 create 创建集群

```
./redis-trib.rb create --replicas 1 192.168.220.128:7000 192.168.220.128:7001 192.168.220.128:7002 192.168.220.128:7003 192.168.220
.128:7004 192.168.220.128:7005
```
其中：--replicas 1参数表示为每个主节点创建一个从节点，其他参数是实例的地址集合。

然后会提示如下：输入yes
```
Can I set the above configuration? (type 'yes' to accept): yes
```
然后显示创建成功，有3个主节点，3个从节点，每个节点都是成功连接状态。

至此redis集群即搭建成功！

另外可对任意节点查看集群节点信息：
```
./redis-trib.rb check 192.168.220.128:7002 #任意一个节点即可
```

## 命令行客户端连接

```
./redis-cli -c -h yourhost -p yourport
```
其中，-c 为指定集群模式，-h,-p分别为主机和端口号。

不指定-c连接集群的会报如下错误：

```
(error) MOVED 1180 192.168.220.128:7003
```


## Redis配置文件分离
使用包含(include)把通用配置和特殊配置分离,方便维护.

#### redis通用配置
```
#GENERAL  
daemonize no  
tcp-backlog 511  
timeout 0  
tcp-keepalive 0  
loglevel notice  
databases 16  
dir /opt/redis/data  
slave-serve-stale-data yes  
#slave只读  
slave-read-only yes  
#not use default  
repl-disable-tcp-nodelay yes  
slave-priority 100  
#打开aof持久化  
appendonly yes  
#每秒一次aof写  
appendfsync everysec  
#关闭在aof rewrite的时候对新的写操作进行fsync  
no-appendfsync-on-rewrite yes  
auto-aof-rewrite-min-size 64mb  
lua-time-limit 5000  
#打开redis集群  
cluster-enabled yes  
#节点互连超时的阀值  
cluster-node-timeout 15000  
cluster-migration-barrier 1  
slowlog-log-slower-than 10000  
slowlog-max-len 128  
notify-keyspace-events ""  
hash-max-ziplist-entries 512  
hash-max-ziplist-value 64  
list-max-ziplist-entries 512  
list-max-ziplist-value 64  
set-max-intset-entries 512  
zset-max-ziplist-entries 128  
zset-max-ziplist-value 64  
activerehashing yes  
client-output-buffer-limit normal 0 0 0  
client-output-buffer-limit slave 256mb 64mb 60  
client-output-buffer-limit pubsub 32mb 8mb 60  
hz 10  
aof-rewrite-incremental-fsync yes  
```
#### redis特殊配置
```
#包含通用配置  
include /opt/redis/redis-common.conf  
#监听tcp端口  
port 6379  
#最大可用内存  
maxmemory 100m  
#内存耗尽时采用的淘汰策略:  
# volatile-lru -> remove the key with an expire set using an LRU algorithm  
# allkeys-lru -> remove any key accordingly to the LRU algorithm  
# volatile-random -> remove a random key with an expire set  
# allkeys-random -> remove a random key, any key  
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)  
# noeviction -> don't expire at all, just return an error on write operations  
maxmemory-policy allkeys-lru  
#aof存储文件  
appendfilename "appendonly-6379.aof"  
#不开启rdb存储,只用于添加slave过程  
dbfilename dump-6379.rdb  
#cluster配置文件(启动自动生成)  
cluster-config-file nodes-6379.conf  
#部署在同一机器的redis实例，把auto-aof-rewrite搓开，因为cluster环境下内存占用基本一致.  
#防止同意机器下瞬间fork所有redis进程做aof rewrite,占用大量内存(ps:cluster必须开启aof)  
auto-aof-rewrite-percentage 80-100  
```
