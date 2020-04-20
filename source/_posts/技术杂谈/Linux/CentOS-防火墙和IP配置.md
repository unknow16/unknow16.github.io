---
title: CentOS-防火墙和IP配置
toc: true
date: 2019-09-04 14:53:05
tags:
categories:
---



## 防火墙
CentOS6关闭防火墙使用以下命令：


```
// 临时关闭
service iptables stop
// 禁止开机启动
chkconfig iptables off
```

CentOS7中若使用同样的命令会报错，这是因为CentOS7版本后防火墙默认使用firewalld，因此在CentOS7中关闭防火墙使用以下命令：

```
// 临时关闭
systemctl stop firewalld
// 禁止开机启动
systemctl disable firewalld

// 输出
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

当然，如果安装了iptables-service，也可以使用下面的命令，

```
// 安装iptables-service
yum install -y iptables-services

//关闭防火墙
service iptables stop

//检查防火墙状态
service iptables status
```

## IP配置
1. 进入IP配置文件目录
```
# CentOS7的目录
/etc/sysconfig/network-scripts
```

2. 通过ifconfig查看网卡名称，CentOS7一般为ens33，CentOS6一般为eth0
3. 编辑相应网卡配置

```
# ens33网卡对应的配置文件为ifcfg-ens33
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"

BOOTPROTO="static"         # 使用静态IP地址，默认为dhcp
IPADDR="192.168.241.100"   # 设置的静态IP地址
NETMASK="255.255.255.0"    # 子网掩码
GATEWAY="192.168.241.2"    # 网关地址
DNS1="192.168.241.2"       # DNS服务器

DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"

NAME="ens33"
UUID="95b614cd-79b0-4755-b08d-99f1cca7271b"
DEVICE="ens33"
ONBOOT="yes"             #是否开机启用

# eth0网卡对应为ifcfg-eth0
```
4. 使用如下名令重启网络即可
```
service network restart
```

5. 如果新装的系统查看没ip
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
可以看到配置最后一项为： ONBOOT=no
表示CentOS 7 默认是不启动网卡的
把这一项改为yes，然后重启网络即可
```
systemctl restart network
```

6. 没有netstat命令
```
yum install net-tools -y
```

## 网卡硬件信息配置
```
# CentOS6的目录位置如下
/etc/udev/rules.d/70-persistent-net.rules
```

## 参考资料
> - []()
> - []()
