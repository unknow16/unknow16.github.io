---
title: Ubuntu-PPA软件源
date: 2018-07-24 16:13:21
tags: Linux
---

### PPA简介
PPA，英文全称为 Personal Package Archives，即个人软件包档案。是 Ubuntu Launchpad 网站提供的一项源服务，允许个人用户上传软件源代码，通过 Launchpad 进行编译并发布为二进制软件包，作为 apt 安装软件时的源供其他用户下载和更新。 

PPA 的一般形式是： ppa:user/ppa-name

### 添加 PPA 源 
1. 添加 PPA 源的命令为：sudo add-apt-repository ppa:user/ppa-name 
2. 添加好记得要更新一下： sudo apt-get update

### 删除 PPA 源 
1. 删除 PPA 源的命令格式则为：sudo add-apt-repository -r ppa:user/ppa-name 
2. 然后进入 /etc/apt/sources.list.d 目录，将相应 ppa 源的保存文件删除。 
3. 最后同样更新一下：sudo apt-get update

----
### 国内更新源
使用如下命令更改(修改前先备份)
```
sudo vim /etc/apt/source.list
```
常用源: https://blog.csdn.net/andersonanya/article/details/77964615



---

## QA
#### apt-get update报错

Q: apt-get update报如下错误？
```
Reading package lists... Done
E: Problem executing scripts APT::Update::Post-Invoke-Success
'if /usr/bin/test -w /var/cache/app-info -a -e /usr/bin/appstreamcli;
 then appstreamcli refresh > /dev/null;
 fi'
E: Sub-process returned an error code
```
A: 解决方法：

三条命令！
```
sudo pkill -KILL appstreamcli

wget -P /tmp https://launchpad.net/ubuntu/+archive/primary/+files/appstream_0.9.4-1ubuntu1_amd64.deb https://launchpad.net/ubuntu/+archive/primary/+files/libappstream3_0.9.4-1ubuntu1_amd64.deb

sudo dpkg -i /tmp/appstream_0.9.4-1ubuntu1_amd64.deb /tmp/libappstream3_0.9.4-1ubuntu1_amd64.deb
```
执行完上述命令之后再次运行sudo apt-get update就不会再出现上面的错误。