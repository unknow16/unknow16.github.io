---
title: Pod生命周期管理
toc: true
date: 2019-08-30 09:09:40
tags:
categories:
---



##  什么是Pod？



## 创建Pod

通常把Pod分为两类：

- 自主式Pod ：

  这种Pod本身是不能自我修复的，当Pod被创建后，都会被Kuberentes调度到集群的Node上。直到Pod的进程终止、被删掉、因为缺少资源而被驱逐、或者Node故障之前这个Pod都会一直保持在那个Node上。Pod不会自愈。如果Pod运行的Node故障，或者是调度器本身故障，这个Pod就会被删除。同样的，如果Pod所在Node缺少资源或者Pod处于维护状态，Pod也会被驱逐。

- 控制器管理的Pod：

  Kubernetes使用更高级的称为Controller的抽象层，来管理Pod实例。Controller可以创建和管理多个Pod，提供副本管理、滚动升级和集群级别的自愈能力。例如，如果一个Node故障，Controller就能自动将该节点上的Pod调度到其他健康的Node上。虽然可以直接使用Pod，但是在Kubernetes中通常是使用Controller来管理Pod的。



```
cat <<EOF >> nginx.conf
 error_log stderr;
 events { worker_connections  1024; }
 http {
     access_log /dev/stdout combined;
     server {
         listen 80 default_server;
         server_name example.com www.example.com;
        location / {
            proxy_pass http://127.0.0.1:2368;
        }
    }
 }
EOF
```

## 参考资料
> - []()
> - []()
