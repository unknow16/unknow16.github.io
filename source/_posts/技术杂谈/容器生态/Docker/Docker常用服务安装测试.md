---
title: Docker常用服务安装测试
toc: true
date: 2019-08-21 16:25:49
tags:
categories:
---



## MySQL-5.7

1. 拉取镜像

```
docker pull mysql:5.7.19
```

2. 创建容器并运行

```
docker run -p 3306:3306 --name mysql-5.7 
-v /data/mysql/log:/var/log/mysql 
-v /data/mysql/data:/var/lib/mysql 
-v /data/mysql/conf:/etc/mysql 
-e MYSQL_ROOT_PASSWORD=root 
-d mysql:5.7.19
```

参数说明 

- -p 3306:3306：将容器的3306端口映射到主机的3306端口
- -v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂在到主机
- -v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
- -v /mydata/mysql/data:/var/lib/mysql/：将数据文件夹挂载到主机
- -e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码



3. 进入运行mysql的docker容器：

```
docker exec -it mysql-5.7  /bin/bash
```

4. 使用mysql命令打开客户端：

```
mysql -uroot -proot --default-character-set=utf8
```

5. 创建mall数据库：

```
create database mall character set utf8
```

6. 安装上传下载插件，并将docment/sql/mall.sql上传到Linux服务器上：

```
yum -y install lrzsz
```

7. 将mall.sql文件拷贝到mysql容器的/目录下：

```
docker cp /mydata/mall.sql mysql:/
```

8. 将sql文件导入到数据库：

```
use mall;
source /mall.sql;
```

9. 创建一个reader帐号并修改权限，使得任何ip都能访问：

```
grant all privileges on *.* to 'reader' @'%' identified by '123456';
```



## Redis-3.2

- 下载redis3.2的docker镜像：

```
docker pull redis:3.2
```

- 使用docker命令启动：

```
  docker run -p 6379:6379 --name redis-3.2 \
  -v /data/redis/data:/data \
  -d redis:3.2 redis-server --appendonly yes
```

- 进入redis容器使用redis-cli命令进行连接：

```
docker exec -it redis-3.2 redis-cli
```

- 测试

```
set a 100
get a
```

 

## Nginx-1.10

1. 下载nginx1.10的docker镜像：

```
docker pull nginx:1.10
```

2. 从容器中拷贝nginx配置

- 先运行一次容器（为了拷贝配置文件）：

```
  docker run -p 80:80 --name nginx-1.10 \
  -v /data/nginx/html:/usr/share/nginx/html \
  -v /data/nginx/logs:/var/log/nginx  \
  -d nginx:1.10
```

- 将容器内的配置文件拷贝到指定目录：

```
docker container cp nginx-1.10:/etc/nginx /data/nginx/
```

- 修改文件名称：

```
mv nginx conf
```

- 终止并删除容器：

```
docker stop nginx-1.10
docker rm nginx-1.10
```

3. 使用docker命令启动：

```
docker run -p 80:80 --name nginx-1.10 \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/logs:/var/log/nginx  \
-v /data/nginx/conf:/etc/nginx \
-d nginx:1.10
```



## RabbitMQ-3.7

1. 下载rabbitmq3.7.15的docker镜像：

```
docker pull rabbitmq:3.7.15
```

2. 使用docker命令启动：

```
  docker run -d --name rabbitmq-3.17 \
  --publish 5671:5671 --publish 5672:5672 --publish 4369:4369 \
  --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 \
  rabbitmq:3.7.15
```

3. 进入容器并开启管理功能：

```
docker exec -it rabbitmq-3.17 /bin/bash
rabbitmq-plugins enable rabbitmq_management
```

4. 登录

```
http://ip:15672

输入账号密码并登录：guest guest
```

5. 创建用户、虚拟host并配置

```
1. 在Admin页面的Users，新增user: mall/mall
2. 在Admin页面的Virtual Hosts，新增虚拟host: /mall
3. 在Admin页面的Users，选中mall用户进入详情配置权限，设置虚拟host为/mall
```



## MongoDB-3.2

1. 下载mongo3.2的docker镜像：

```
docker pull mongo:3.2
```

2. 使用docker命令启动：

```
  docker run -p 27017:27017 --name mongo-3.2 \
  -v /data/mongo/db:/data/db \
  -d mongo:3.2
```




## ElasticSearch-6.4

1. 下载elasticsearch6.4.0的docker镜像：

```
docker pull elasticsearch:6.4.0
```

2. 修改虚拟内存区域大小，否则会因为过小而无法启动:

```
sysctl -w vm.max_map_count=262144
```

3. 使用docker命令启动

```
docker run  --name elasticsearch-6.4 \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:6.4.0
```

4. 启动时会发现/usr/share/elasticsearch/data目录没有访问权限，只需要修改/mydata/elasticsearch/data目录的权限，再重新启动。

```
chmod 777 /mydata/elasticsearch/data/
```

5. 测试

```
http://ip:9200
```

- 在线安装中文分词器IKAnalyzer

```
# 1. 进入容器
docker exec -it elasticsearch-6.4 /bin/bash
# 2. 在容器中安装
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.0/elasticsearch-analysis-ik-6.4.0.zip
# 3. 重启
docker restart elasticsearch
```

- 离线安装中文分词器IKAnalyzer

```
# 1. 进入插件目录，新建elasticsearch子目录，并进入
cd /data/elasticsearch/plugins
mkdir elasticsearch
cd elasticsearch
# 2. 下载插件
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.0/elasticsearch-analysis-ik-6.4.0.zip
# 3. 解压
unzip elasticsearch-analysis-ik-6.4.0.zip
# 4. 删除源zip包
rm -f elasticsearch-analysis-ik-6.4.0.zip
```



## Kinana-6.4

- 下载kibana6.4.0的docker镜像：

```
docker pull kibana:6.4.0
```

- 使用docker命令启动：

```
  docker run --name kibana-6.4 -p 5601:5601 \
  --link elasticsearch-6.4:elasticsearch \
  -e "elasticsearch.hosts=http://elasticsearch:9200" \
  -d kibana:6.4.0
```

- 测试：

```
http://ip:5601
```



## Nexus-3.x

Nexus3.x支持Docker、Maven、Yum、PyPI等私有仓库。

```
docker run -d --name nexus3 --restart=always 
-p 8081:8081 
--mount src=nexus-data,target=/nexus-data 
sonatype/nexus3
```

默认访问地址：<http://ip:8081>

默认账号密码：admin/admin123



## 参考资料
> - []()
> - []()
