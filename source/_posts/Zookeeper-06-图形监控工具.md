---
title: Zookeeper-06-图形监控工具
date: 2018-06-13 15:24:28
tags: Zookeeper
---

## ZooInspector 
1. 下载:

```
https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip
```
2. 解压运行，需java环境

```
// 1. 双击运行
ZooInspector\build\zookeeper-dev-ZooInspector.jar

// 2. 采用命令运行
java -jar zookeeper-dev-ZooInspector.jar
```
## Eclipse插件 

* 安装插件
1. 在 Eclipse  菜单打开Help -> Install New Software... 
2. 添加 url http://www.massedynamic.org/eclipse/updates/ . 
3. 选择插件并安装 

* 运行
1. 在 Eclipse  菜单打开Window->Show View->Other...->ZooKeeper 
2. 连接ZK，输入正在运行的ZK server 地址和端口 
3. 连接成功后就就可以在Eclipse里查看ZK Server里的节点信息.

## taokeeper
github: https://github.com/alibaba/taokeeper

- CPU/MEM/LOAD的监控
- ZK日志目录所在磁盘剩余空间监控
- 单机连接数的峰值报警
- 单机 Watcher数的峰值报警
- 节点自检：是指对集群中每个IP所在ZK节点上的PATH: /YINSHI.MONITOR.ALIVE.CHECK 定期进行三次如下流程 : 节点连接 – 数据发布 – 修改通知 – 获取数据 – 数据对比, 在指定的延时内，三次流程均成功视为该节点处于正常状态。
- ZooKeeper集群实时状态