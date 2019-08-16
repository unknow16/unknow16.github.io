---
title: 03-Nginx的反向代理配置
toc: true
date: 2019-07-15 11:15:26
tags:
categories:
---


1. 新建反向代理服务器配置文件conf/reverse-proxy.conf
2. 在conf/nginx.conf中的http块中包含conf/reverse-proxy.conf

```
include  reverse-proxy.conf;
```



## 反向代理配置示例

```
server {
    listen 80;
    server_name nexus.minfy.cn;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:36006;
    }
    access_log logs/nexus.minfy.cn.log;
}
 
server {
    listen 80;
    server_name jenkins.minfy.cn;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:36008;
    }
    access_log logs/jenkins.minfy.cn.log;
}
```



## 负载均衡配置

```
upstream monitor_server {
    server 192.168.0.131:80;
    server 192.168.0.132:80;
}
 
server {
    listen 80;
    server_name jenkins.minfy.cn;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_pass http://monitor_server;
    }
    access_log logs/jenkins.minfy.cn.log;
}
```



## 经常504超时解决方法

原因是超时，参数相应调大即可解决，均为http块下配置

```
proxy_connect_timeout 300s; # nginx和后端服务器建立连接的超时时间
proxy_read_timeout 300s; # 连接成功后，后端服务器响应超时时间
proxy_send_timeout 300s; 
proxy_buffer_size 64k; # nginx保存用户头信息的缓冲区大小
proxy_buffers 4 32k; 
proxy_busy_buffers_size 64k; # 高负荷下缓存大小
proxy_temp_file_write_size 64k; 
proxy_ignore_client_abort on; # 不允许nginx主动关闭连接
```





## 参考资料
> - []()
> - []()
