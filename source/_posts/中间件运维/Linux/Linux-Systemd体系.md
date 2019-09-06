---
title: Linux-03-Systemd体系
date: 2018-07-24 16:12:47
tags: Linux
---

## Systemd 体系
Systemd是Linux下的一种init软件，Systemd目的是要取代Unix时代以来一直在使用的init系统，兼容SysV和LSB的启动脚本，而且够在进程启动过程中更有效地引导加载服务。 

在CentOS-7系统中，pid为1被systemd进程所使用，CentOS-7之前pid为1的进程是init进程。其他进程都是通过system或init进程启动的，即都是它们的子进程，可通过pstree -p命令查看进程树结构，-p参数指定显示每个进程的pid。

CentOS-7起开始采用systemctl命令控制系统服务。

## Systemd 新概念
在systemd的管理体系里面，以前的运行级别（runlevel）的概念被新的运行目标（target）所取代。

tartget的命名类似于multi-user.target等这种形式，比如原来的运行级别3（runlevel3）就对应新的多用户目标（multi-user.target），run level 5就相当于graphical.target。 

在systemd的管理体系里面，默认的target的是default.target，相当于以前的默认运行级别，是通过软链到graphical.target来实现。

在Systemd体系中也定义了runlevelX.target文件目的主要是为了能够兼容以前的运行级别level的管理方法。 事实上runlevelX.target同样是被软连接到相应target。映射关系如下：

```
runlevel0.target -> poweroff.target
runlevel1.target -> rescue.target
runlevel2.target -> multi-user.target
runlevel3.target -> multi-user.target
runlevel4.target -> multi-user.target
runlevel5.target -> graphical.target
runlevel6.target -> reboot.target
```


## Systemd 服务脚本存放目录
```
# CentOS-7: 
/usr/lib/systemd/system

# Ubuntu-16: 
/lib/systemd/system
```

## Systemd 基本工具
Systemd使用systemctl命令管理。 

- 使服务自启动 systemctl enable xxx.service
- 使服务自动禁止 systemctl disable xxx.service
- 检查服务状态 systemctl status xxx.service
- 启动某服务 systemctl start xxx.service
- 停止某服务 systemctl stop xxx.service
- 重启某服务 systemctl restart xxx.service









