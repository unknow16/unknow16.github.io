---
title: Linux-04-Systemd下配置tomcat服务
date: 2018-07-24 16:12:59
tags: Linux
---

该配置案例是要CentOS-7中进行的。

#### 1. 新建tomcat.service

在/usr/lib/systemd/system下新建tomcat.service,内容如下,下面ExecStart和ExecStop后面的路径要根据实际情况修改。
```
[Unit]  
Description=tomcatapi  
After=network.target  
   
[Service]  
Type=forking  
ExecStart=/usr/local/soft/tomcat/tomcat8/bin/startup.sh  
ExecReload=  
ExecStop=/usr/local/soft/tomcat/tomcat8/bin/shutdown.sh  
PrivateTmp=true  
   
[Install]  
WantedBy=multi-user.target 
```

#### 2. 给这个tomcat.service 文件添加可执行权限

```
chomod +x tomcat.service
```

#### 3. 重启systemctl
```
systemctl daemon-reload
```
#### 4. 使用systemctl启动tomcat

```
systemctl start tomcat
```
