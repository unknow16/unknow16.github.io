---
title: 02-Nginx的虚拟主机配置
toc: true
date: 2019-07-15 11:15:09
tags:
categories:
---



虚拟主机配置区分不同的网站有三种方式：

1. 域名区分
2. 端口区分(不常用)
3. ip区分(不常用)



## 基于域名区分配置

1. 一般情况下会在conf下新建一 个vhost目录

2. 然后在vhost中每个二级域名一个配置文件

3. 在conf/nginx.conf中http块中将vhost下配置文件包含进来

   ```
   include  vhost/*.conf;
   ```



- vhost/a.minfy.cn.conf 配置文件

```
server {
    listen       80;
    server_name  a.minfy.cn;

    location / {
        root   /usr/local/nginx-1.14/data/html-a;
        index  index.html index.htm;
    }
    
    access_log logs/a.minfy.cn.log;
}
```

- vhost/b.minfy.cn.conf 配置文件

```
server {
    listen       80;
    server_name  b.minfy.cn;

    location / {
        root   /usr/local/nginx-1.14/data/html-b;
        index  index.html index.htm;
    }
    
    access_log logs/b.minfy.cn.log;
}
```



## 参考资料
> - []()
> - []()
