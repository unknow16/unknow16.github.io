---
title: Ubuntu-常用设置
toc: true
date: 2020-01-07 13:55:06
tags:
categories:

---



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