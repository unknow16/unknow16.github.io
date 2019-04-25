---
title: Redis-01-单机安装
date: 2018-06-22 18:42:22
tags: Redis
---


#### 下载redis --> 解压 --> 进入解压目录 --> 编译源码

```
$ wget http://download.redis.io/releases/redis-3.2.9.tar.gz
$ tar zxvf redis-3.2.9.tar.gz 
$ cd redis-3.2.9
$ make install PREFIX=/usr/local/redis-3.2.9-bin（安装目录）
```
1. 完成安装后，会在安装目录下生成一个bin目录，bin目录下包含如下可执行文件：
```
redis-benchmark ： 用于测试redis的性能。
redis-check-aof : 当aof备份文件被损坏，可通过该工具对aof文件进行修复，使用方式：redis-check-aof --fix 要修复的aof文件。
redis-check-rdb : 修复损坏的rdb备份文件。
redis-cli : redis客户端，用于连接服务端。
redis-server ： redis服务器端，用于启动redis服务器。
redis-sentinel : 哨兵模式（实际使用较多） 在master-slave模式下（slave默认不支持写），当master出现异常时，自动在slave中选择一台作为master。
```
2. 然后将redis-3.2.9/redis.conf复制到redis-3.2.9-bin下
3. 修改配置文件,使其后台启动
```
#查找daemonize no改为 yes
#以守护进程方式运行  
daemonize yes 
```


#### 启动redis  

```
$ cd /usr/local/redis-3.2.9-bin
$ ./redis-server --help # 查看命令参数选项
$ ./redis-server redis.conf # 以指定配置文件启动

$ ./redis-cli shutdown # 关闭redis服务
```
  
#### 查看redis是否己启动  

```
$ ps -ef | grep redis
```

#### 连接测试

```
$ cd /usr/local/redis-3.2.9-bin
$ ./redis-cli --help # 查看命令参数选项

$ ./redis-cli -h localhost # 连接本地redis服务
localhost:6379>set foo bar # 设置key-value
localhost:6379>get foo  # 根据key获取value
```

用redis-cli连接上redis服务器后，可通过指令“info”查看redis服务器信息，也可查看服务器知道内容信息，例如：info replication 查看主从相关信息。

```
localhost:6379> info # 查看redis服务器信息
localhost:6379> info replication # 查看主从相关信息
```

## 配置外网访问
#### 1. 将绑定的本机给注释掉
redis.conf 中的 bind 127.0.0.1注释掉，前加#号

#### 2. 设置redis-cli连接redis服务器的密码
redis.conf 中的 requirepass xxx 注释解开

xxx为可自行设置客户端连接时的密码

#### 3. 开放redis防火墙端口

```
# 关闭防火墙  
service iptables stop

# 开放6379端口
vim /etc/sysconfig/iptables

# 添加  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6379 -j ACCEPT 

# 重启防火墙  
service iptables restart
```

#### 4. 重启redis服务
```
$ ./redis-server redis.conf # 以指定配置文件启动
```

#### 5. 客户端连接测试

```
$ ./redis-cli -h localhost -p 6379 -a xxx
```
-h 是连接的主机ip，host的缩写

-p 是端口 port的缩写

-a 后面是密码（requirepass 后面配置的）auth的缩写



## 将redis添加到自启动中  

```
echo "/usr/local/redis-3.2.9-bin/redis-server /usr/local/redis-3.2.9-bin/redis.conf" >> /etc/rc.d/rc.local
```
