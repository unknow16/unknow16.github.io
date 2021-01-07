---
title: 03-kubectl常用命令
toc: true
date: 2019-08-29 11:00:00
tags:
categories:
---

```
# 查看资源对象详细配置
kubectl get pod myapp-848b5b879b-5f69p -o yaml
kubectl describe pods nginx-554b9c67f9-f2mwt

# 标签选择查看pod
kubectl get pods -l app=filebeat -o wide
kubectl get pods -w　　# 观察更新状态

## 日志查看
kubectl logs -f  pod-demo
kubectl logs pod-demo myapp # 查看pod中myapp容器的日志

## 进入pod中的myapp容器中
kubectl exec -it pod-demo  -c myapp -- /bin/sh

# 查看资源对象配置文档
kubectl explain pod
kubectl explain pod.spec

# 查看集群相关信息
kubectl get node
kubectl get cs
kubectl api-resources
kubectl api-versions
```

## ingress相关

```
# 查看当前命名空间下ingress对象列表
kubectl get ingress

# 查看当前命名空间下的所有ingress详细规则
kubectl describe ingress

# 查看当前命名空间下指定的tomcat-ingress的规则
kubectl describe ingress tomcat-ingress

# 进入ingress-controller的pod中查看规则
kubectl exec -n ingress-nginx -it nginx-ingress-controller-6bd7c597cb-6pchv -- /bin/bash
```





## 参考资料
> - []()
> - []()
