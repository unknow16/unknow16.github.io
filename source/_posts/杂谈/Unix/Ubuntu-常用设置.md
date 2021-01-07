---
title: Ubuntu-常用设置
toc: true
date: 2020-01-07 13:55:06
tags:
categories:

---


## sudo找不到命令
问题：普通用户下，设置并export一个变量，然后利用sudo执行echo命令，能得到变量的值，但是如果把echo命令写入脚本，然后再sudo执行脚本，就找不到变量，未能获取到值
原因：sudo运行时，会默认重置环境变量为安全的环境变量(如下的secure_path)，也即，但前面设置的变量都会失效，只有少数配置文件中指定的环境变量能保存下来。sudo的配置文件是 /etc/sudoers 需要root权限才能读取:
```
Defaults env_reset 
Defaults mail_badpass 
Defaults secure_path=”/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin” 

root ALL=(ALL:ALL) ALL 
%sudo ALL=(ALL:ALL) ALL 
xxx ALL=(ALL:ALL) NOPASSWD:ALL
```
也可以直接通过sudo -l来查看sudo的限制，注意看第一行的选项 Defaults env_reset 表示默认会将环境变量重置，这样你定义的变量在sudo环境就会失效，获取不到。另外有的发行版还有一个Defaults env_keep=”“的选项，用于保留部分环境变量不被重置，需要保留的变量就写入双引号中。

解决：
1. sudo -E:  E选项在man page中的解释是：简单来说，用户可以在sudo执行时保留当前用户已存在的环境变量，不会被sudo重置，另外，如果用户对于指定的环境变量没有权限，则会报错。
2. 修改sudo配置文件: 在内部测试机器中，安全性要求不高，总是需要加上-E参数来执行脚本，这个安全设定也不是很方便，可以注释掉默认配置env_reset和env_keep


## PPA简介

Ubuntu 提供了一个名为 Launchpad 的平台，使软件开发人员能够创建自己的软件仓库。终端用户可以将 PPA 仓库添加到 `/etc/apt/sources.list` 文件中，然后可以通过`apt-get`安装它。

- Launchpad官网：<https://launchpad.net/>

> Q: apt-get update报如下错误？

```
Reading package lists... Done
E: Problem executing scripts APT::Update::Post-Invoke-Success
'if /usr/bin/test -w /var/cache/app-info -a -e /usr/bin/appstreamcli;
 then appstreamcli refresh > /dev/null;
 fi'
E: Sub-process returned an error code
```

A: 解决方法：三条命令！

```
sudo pkill -KILL appstreamcli

wget -P /tmp https://launchpad.net/ubuntu/+archive/primary/+files/appstream_0.9.4-1ubuntu1_amd64.deb https://launchpad.net/ubuntu/+archive/primary/+files/libappstream3_0.9.4-1ubuntu1_amd64.deb

sudo dpkg -i /tmp/appstream_0.9.4-1ubuntu1_amd64.deb /tmp/libappstream3_0.9.4-1ubuntu1_amd64.deb
```

执行完上述命令之后再次运行sudo apt-get update就不会再出现上面的错误。