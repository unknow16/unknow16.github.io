---
title: Linux-01-SystemV引导启动流程
date: 2018-07-24 16:10:41
tags: Linux
---

#### 1. BIOS
#### 2. /boot/grub/grub.conf
此配置文件中指定以下信息

- kernel : 指定找哪一个内核
- initrd : 内核的引导文件

#### 3. 根据引导文件启动 kernel
#### 4. 由内核加载运行init进程

- init进程：/sbin/init,是系统的第一个进程,PID=1

#### 5. startup阶段

- init进程启动后发出 “startup” 事件，
- 事件驱动init进程调用并加载 /etc/init/rcS.conf
- rcS.conf 又加载 /etc/rc.d/rc.sysinit和/etc/inittab
- send “runlevel” event to init

#### 6. runlevel阶段

- 事件驱动init进程调用并加载 /etc/init/rc.conf
- rc.conf 又加载 /etc/rc.d/rc[0-6].d/
- send "rc" event to init

#### 7. rc阶段

- 根据运行级别启动相应的图形界面或终端
- /etc/init/start-ttys.conf
- /etc/init/prefdm.conf

#### 8. 启动完成

---

## 各配置文件功能
#### /etc/rc.d/rc.sysinit 
* 完成主机的名称，网络，文件系统，挂载逻辑卷，设置时间等

#### /etc/inittab
* 选择运行级别

#### /etc/rc.d/rc[0-6].d/
* /etc/rc.d/init.d/ : 包含所有服务启动脚本
* /etc/rc.d/rc5.d/  : 其中包含在运行级别5时，要启动的服务的脚本的快捷方式，指向init.d/下的服务脚本
* 其中服务名，S-前缀为在此运行级别开机启动的服务，K-前缀为开机不启动

#### /etc/rc.d/rc
* 由init进程调用执行
* 根据指定的运行级别，加载或终止相应的系统服务

#### /etc/rc.d/rc.local
* 由rc脚本调用
* 保存用户定义的需卡机自动执行的命令