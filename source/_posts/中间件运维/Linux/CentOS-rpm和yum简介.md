---
title: CentOS-rpm和yum简介
toc: true
date: 2019-09-04 14:52:45
tags:
categories:
---



## RPM诞生
最早期时，软件包是一些可以运行的程序组成的集合，可能还要加上若干配置文件和动态库。例如，程序员将针对某个平台编译好的二进制文件、程序所依赖的动态库文件（如 .so 和 .dll 为扩展名的文件）以及配置文件复制到一个目录中，这个目录就可以称为一个软件包。

为了保证使用的软件包能够方便且快速地复制到别的机器上， 人们开始选用压缩文件的方式来封装软件包。 比如通过 tar 或者 gzip 压缩后得到 .tar.gz、 .rar 或者 .zip 格式的文件， 这时我们就获得了一个较为高级的软件包。

再往后发展， 就出现了更高级的软件包， 比如 .rpm、 .bin 或者 .deb 格式的软件包。 这些格式的软件包， 相对于压缩格式的软件包又有了更进一步的发展， 它们不仅支持文件压缩功能， 还有依赖维护、 脚本的嵌入等功能。RedHat 公司开发贡献的 RedHat Package Manager（RPM） 可以说是这些高级别软件包中最典型的一个。

## RPM 包的命名方式

以 httpd-2.2.15-39.el6.centos.x86_64.rpm 为例：
- httpd 表示软件名
- 2.2.15 分别表示主版本号、次版本号、发行版本号
- 39.el6.centos 表示 RPM 包的修订号和 OS 信息
- x86_64 表示此软件包适用的平台，常见的有i386，i586，x86_64

## RPM常用命令组合

```
// 安装显示安装进度  --install--verbose--hash
rpm -ivh *.rpm

// 升级软件包  --Update
rpm －Uvh *.rpm

// 搜索指定rpm包是否安装  --all
rpm -qa | grep ssh

// 列出所有文件安装目录 --list
// 此处需指定具体软件包名，可通过qa模糊查询
rpm -ql openssh-server-7.4p1-16.el7.x86_64

// 删除一个rpm包   --erase
rpm -e file.rpm      
```

## yum 简介
yum，是Yellow dog Updater, Modified 的简称，是杜克大学为了提高RPM 软件包安装性而开发的一种软件包管理器。起初是由yellow dog 这一发行版的开发者Terra Soft 研发，用python 写成，那时还叫做yup(yellow dog updater)，后经杜克大学的Linux@Duke 开发团队进行改进，遂有此名。yum 的宗旨是自动化地升级，安装/移除rpm 包，收集rpm 包的相关信息，检查依赖性并自动提示用户解决。yum 的关键之处是要有可靠的repository，顾名思义，这是软件的仓库，它可以是http 或ftp 站点，也可以是本地软件池，但必须包含rpm 的header，header 包括了rpm 包的各种信息，包括描述，功能，提供的文件，依赖性等。正是收集了这些header 并加以分析，才能自动化地完成余下的任务。

yum 的理念是使用一个中心仓库(repository)管理一部分甚至一个distribution 的应用程序相互关系，根据计算出来的软件依赖关系进行相关的升级、安装、删除等等操作，减少了Linux 用户一直头痛的dependencies 的问题。这一点上，yum 和apt 相同。apt 原为debian 的deb 类型软件管理所使用，但是现在也能用到RedHat 门下的rpm 了。

## yum 配置
yum 的配置文件分为两部分：main 和repository

- main 部分定义了全局配置选项，整个yum 配置文件应该只有一个main。常位于/etc/yum.conf 中。
- repository 部分定义了每个源/服务器的具体配置，可以有一到多个。常位于/etc/yum.repo.d 目录下的各文件中。

## 国内镜像
这些镜像站提供了包括 Arch Linux, CentOS, CPAN, CTAN, Cygwin, Debian, 
  Deepin, EPEL, Fedora, Gentoo, Kali Linux, Linux Mint, NeuroDebian, 
  openSUSE, PostgreSQL, Ubuntu 等项目源的镜像

- 搜狐开源镜像站：http://mirrors.sohu.com/
- 网易开源镜像站：http://mirrors.163.com/
- 阿里云镜像： mirrors.aliyun.com
- 
- 北京理工大学：
- http://mirror.bit.edu.cn (IPv4 only)
- http://mirror.bit6.edu.cn (IPv6 only)
- 
- 北京交通大学：
- http://mirror.bjtu.edu.cn (IPv4 only)
- http://mirror6.bjtu.edu.cn (IPv6 only)
- http://debian.bjtu.edu.cn (IPv4+IPv6)
- 
- 兰州大学：http://mirror.lzu.edu.cn/
- 厦门大学：http://mirrors.xmu.edu.cn/
- 
- 清华大学：
- http://mirrors.tuna.tsinghua.edu.cn/ (IPv4+IPv6)
- http://mirrors.6.tuna.tsinghua.edu.cn/ (IPv6 only)
- http://mirrors.4.tuna.tsinghua.edu.cn/ (IPv4 only)
- 
- 天津大学：http://mirror.tju.edu.cn/
- 
- 中国科学技术大学：
- http://mirrors.ustc.edu.cn/ (IPv4+IPv6)
- http://mirrors4.ustc.edu.cn/
- http://mirrors6.ustc.edu.cn/
- 
- 东北大学：
- http://mirror.neu.edu.cn/ (IPv4 only)
- http://mirror.neu6.edu.cn/ (IPv6 only)
- 
- 电子科技大学：http://ubuntu.uestc.edu.cn/
- 浙江大学： http://mirrors.zju.edu.cn
- CentOS镜像列表：https://www.centos.org/download/mirrors/

## 参考资料
> - []()
> - []()
