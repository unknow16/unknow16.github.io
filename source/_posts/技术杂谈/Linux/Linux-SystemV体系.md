---
title: Linux-SystemV体系
date: 2018-07-24 16:12:32
tags: Linux
---

## System V体系
发展到今天，大多数基于Linux的操作系统，使用的是System-V风格的init守护进程，换句话说，它们的启动处理由init进程管理，其管理功能在一定程度上继承了基于System V 的Unix操作系统。

该守护进程根据运行级别(run level)的原则，系统的运行级别表示当前计算机的状态。

## System V的7个运行级别含义
- runlevel-0: 系统停机状态，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。 
- runlevel-1: 单用户工作状态，root权限，用于系统维护，禁止远程登陆，就像Windows下的安全模式登录。 
- runlevel-2: 多用户状态，没有NFS支持。 
- runlevel-3: 完整的多用户模式，有NFS，登陆后进入控制台命令行模式。 
- runlevel-4: 系统未使用，保留一般不用，在一些特殊情况下可以用它来做一些事情。例如在笔记本电脑的电池用尽时，可以切换到这个模式来做一些设置。 
- runlevel-5: X11控制台，登陆后进入图形GUI模式，XWindow系统。 
- runlevel-6: 系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动。运行init 6机器就会重启。

标准的Linux运行级别为3或5

## System V的基本工具
system V主要用chkconfig/sevice/update-rc.d命令管理服务，在使用这些命令操作服务前，需要将相应服务脚本放入/etc/init.d目录中。 

chkconfig基本命令如下： 

- 添加服务 chkconfig –add servicename 
- 使服务自动启动 chkconfig –level 2345 servicename on 
- 使服务自动禁止 chkconfig –level 2345 servicename off 
- 删除服务 chkconfig –del servicename 
- 检查服务状态 chkconfig servicename status 
- 显示所有已启动的服务 chkconfig –list 

service基本命令如下： 

- 启动某服务 service servicename start 
- 停止某服务 service servicename stop 
- 重启某服务 service servicename restart 

update-rc.d基本命令如下： 

- 删除服务 update-rc.d -f servicename remove