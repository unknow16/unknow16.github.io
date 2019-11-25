---
title: fluentd安装
toc: true
date: 2019-11-20 14:59:15
tags:
categories:
---



## 准备工作

1. 同步时间，对日志服务极其重要
2. 请增加文件描述符的最大数量。您可以使用ulimit -n命令检查当前数量。如果您的控制台显示1024，则它不足。请在/etc/security/limits.conf文件中添加以下行，然后重新启动计算机。

```
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
```



## td-agent和Fluentd分别是什么

Fluentd是用Ruby编写的，以提高灵活性，而性能敏感的部分则用C编写。但是，某些用户可能难以安装和操作Ruby守护程序。这就是Treasure Data公司提供稳定的Fluentd发行版本的原因，该发行版本称为td-agent。 Fluentd和td-agent之间的差异可以在这里找到。

- <https://www.fluentd.org/faqs>

本安装指南适用于新的稳定版本td-agent v3。 td-agent v3在核心中使用fluentd v1.0。

## 参考资料
> - []()
> - []()
