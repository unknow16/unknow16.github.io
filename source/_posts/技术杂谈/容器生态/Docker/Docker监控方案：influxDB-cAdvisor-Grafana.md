---
title: Docker监控方案：influxDB+cAdvisor+Grafana
toc: true
date: 2019-08-27 15:20:12
tags:
categories:
---



cAdvisor：用于数据采集，是 Google用来监控他们基础设施的一款工具，这个工具厉害之处不仅能监控docker容器的实时信息，而且还能将你的cadvisor这容器所在的主机的系统的实时信息，但是由于cAdvisor只是能监控到实时的信息而不能保存。

influxDB： 用于监控数据存储，是一个分布式时间序列数据库。

Grafana： 用户可视化监控数据，有着非常漂亮的图表和布局展示，功能齐全的度量仪表盘和图形编辑器。支持Graphite、zabbix、InfluxDB、Prometheus和 OpenTSDB作为数据源。 



## influxDB安装

1. 创建服务


```
docker run -d --name influxdb -p 8086:8086 -p 8083:8083 tutum/influxdb
```

参数说明：

```
-d ：后台运行此容器
--name ：启运容器分配名字influxdb
-p 8083:8083 ： 8083端口为infuxdb管理端口
-p 8086:8086  : 8086端口是infuxdb的数据端口
```

2. 通过ip:8083访问管理端
3. 在管理端创建数据库和用户

```
创建数据库:  CREATE DATABASE "cadvisor"
查看数据库:  SHOW DATABASES  

创建用户并授权: CREATE USER "cadvisor" WITH PASSWORD 'cadvisor' WITH ALL PRIVILEGES
查看用户: SHOW USRES
```

4. 用户授权

```
grant all privileges on cadvisor to cadvisor 
grant WRITE on cadvisor to cadvisor 
grant READ on cadvisor to cadvisor
```

管理端右上切换到cadvisor数据库，使用以下命令查看采集的数据

```
SHOW MEASUREMENTS
```

现在我们还没有数据，如果想采集系统的数据，我们需要使用**cAdvisor**软件来实现

## **cAdvisor**安装

1. 创建服务

```
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro --publish=8082:8080 --detach=true \
--link influxdb:influxdb --name=cadvisor google/cadvisor:latest \
-storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxdb:8086
```

参数说明：

```
-d ：后台运行此容器
--name ：启运容器分配名字cadvisor
-p ：映射端口8082:8080（由于环境8080端口被占用，cadvisor默认端口是8080不建议修改）
-v：把宿主机的目录映射到容器中，这些目录都是cadvisor需要采集的目录文件和监控内容
-storage_driver：需要指定cadvisor的存储驱动、数据库主机、数据库名
```

2. 通过ip:8082访问cAdvisor管理页面，cAdvisor的基础图形功能也酷炫的



## Granafa安装

1. 创建服务

```
docker run -d --name grafana -p 3000:3000 grafana/grafana
```

2. 访问WEB管理端，默认用户名，密码 admin/admin



## 参考资料
> - []()
> - []()
