---
title: 01-Linux安装nodejs和npm
toc: true
date: 2018-04-19 20:02:35
tags: Linux
---

### 
nodejs是一个基于Chrome V8引擎的JavaScript运行环境，其使用了一个事件驱动、非阻塞式I/O模型。

npm是全球最大的开源库生态系统。

### 安装

```
//1. 一般安装在/usr/local目录下
cd /usr/local

//2. 下载二进制包
wget https://nodejs.org/dist/v10.15.3/node-v10.15.3-linux-x64.tar.xz

//3. 解压
tar -xvf node-v10.15.3-linux-x64.tar.xz

//4. 根据不同系统配置环境变量
export NODE_HOME=/node-v10.15.3-linux-x64
export PATH=$PATH:$NODE_HOME/bin 
export NODE_PATH=$NODE_HOME/lib/node_modules

//5. 使配置生效

```

### 配置npm仓库
国内网络环境从npm官方源安装软件包速度会比较慢，甚至导致安装不成功。

可以安装nrm工具，用于管理软件源。
```
npm install -g nrm

//安装完成之后，列出可用的软件源
$ nrm ls
* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
  
//在国内，我们可以使用taobao的源，速度还相对不错。

$ nrm use taobao

Registry has been set to: https://registry.npm.taobao.org/
```

### n管理器
用于管理nodejs版本

```
//安装n
sudo npm install n -g

//帮助命令
sudo n -h

//输出所有可用版本号
sudo n ls

//安装并使用指定版本
sudo n 8.11.0
```



##### Ubuntu 16.04出现：Problem executing scripts APT::Update::Post-Invoke-Success 'if /usr/bin/test -w /var/cache/app-info -a -e /usr/bin/appstreamcli; then appstreamcli refresh > /dev/null; fi'

* 错误

```
Reading package lists... Done
E: Problem executing scripts APT::Update::Post-Invoke-Success
'if /usr/bin/test -w /var/cache/app-info -a -e /usr/bin/appstreamcli;
 then appstreamcli refresh > /dev/null;
 fi'
E: Sub-process returned an error code
```
* 在运行sudo apt-get update时出现如上信息，解决方法如下：
```
sudo pkill -KILL appstreamcli

wget -P /tmp https://launchpad.net/ubuntu/+archive/primary/+files/appstream_0.9.4-1ubuntu1_amd64.deb https://launchpad.net/ubuntu/+archive/primary/+files/libappstream3_0.9.4-1ubuntu1_amd64.deb

sudo dpkg -i /tmp/appstream_0.9.4-1ubuntu1_amd64.deb /tmp/libappstream3_0.9.4-1ubuntu1_amd64.deb
```
执行完上述命令之后再次运行sudo apt-get update就不会再出现上面的错误。