---
title: 10-存储卷：configMap
toc: true
date: 2019-09-08 21:42:55
tags:
categories:
---



## ConifgMap解析

configmap是让配置文件从镜像中解耦，让镜像的可移植性和可复制性。许多应用程序会从配置文件、命令行参数或环境变量中读取配置信息。这些配置信息需要与docker  image解耦，你总不能每修改一个配置就重做一个image吧？ConfigMap  API给我们提供了向容器中注入配置信息的机制，ConfigMap可以被用来保存单个属性，也可以用来保存整个配置文件或者JSON二进制大对象。

ConfigMap API资源用来保存key-value  pair配置数据，这个数据可以在pods里使用，或者被用来为像controller一样的系统组件存储配置数据。虽然ConfigMap跟Secrets类似，但是ConfigMap更方便的处理不含敏感信息的字符串。   注意：ConfigMaps不是属性配置文件的替代品。ConfigMaps只是作为多个properties文件的引用。可以把它理解为Linux系统中的/etc目录，专门用来存储配置文件的目录。下面举个例子，使用ConfigMap配置来创建Kuberntes  Volumes，ConfigMap中的每个data项都会成为一个新文件。



```
[root@k8s-master volumes]# kubectl explain cm
KIND:     ConfigMap
VERSION:  v1
FIELDS:
   apiVersion   <string>
   binaryData   <map[string]string>
   data <map[string]string>
   kind <string>
   metadata <Object>
```

## ConfigMap创建方式
与 Secret 一样，ConfigMap 也支持四种创建方式：

1. 通过 --from-literal：每个 --from-literal 对应一个信息条目

```
[root@k8s-master volumes]# kubectl create configmap nginx-config --from-literal=nginx_port=80 --from-literal=server_name=myapp.magedu.com
configmap/nginx-config created
[root@k8s-master volumes]# kubectl get cm
NAME           DATA      AGE
nginx-config   2         6s
[root@k8s-master volumes]# kubectl describe cm nginx-config
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
server_name:
----
myapp.magedu.com
nginx_port:
----
80
Events:  <none>
```



2. 通过 --from-file：每个文件内容对应一个信息条目

```
[root@k8s-master mainfests]# mkdir configmap && cd configmap
[root@k8s-master configmap]# vim www.conf
server {
    server_name myapp.magedu.com;
    listen 80;
    root /data/web/html;
}
[root@k8s-master configmap]# kubectl create configmap nginx-www --from-file=./www.conf 
configmap/nginx-www created
[root@k8s-master configmap]# kubectl get cm
NAME           DATA      AGE
nginx-config   2         3m
nginx-www      1         4s
[root@k8s-master configmap]# kubectl get cm nginx-www -o yaml
apiVersion: v1
data:
  www.conf: "server {\n\tserver_name myapp.magedu.com;\n\tlisten 80;\n\troot /data/web/html;\n}\n"
kind: ConfigMap
metadata:
  creationTimestamp: 2018-10-10T08:50:06Z
  name: nginx-www
  namespace: default
  resourceVersion: "389929"
  selfLink: /api/v1/namespaces/default/configmaps/nginx-www
  uid: 7c3dfc35-cc69-11e8-801a-000c2972dc1f
```

## 如何使用configMap？
1. 环境变量方式注入到pod

```
[root@k8s-master configmap]# vim pod-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-1
  namespace: default
  labels: 
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80 
    env:
    - name: NGINX_SERVER_PORT
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: nginx_port
    - name: NGINX_SERVER_NAME
      valueFrom:
        configMapKeyRef:
          name: nginx-config
          key: server_name
[root@k8s-master configmap]# kubectl apply -f pod-configmap.yaml 
pod/pod-cm-1 created
[root@k8s-master configmap]# kubectl exec -it pod-cm-1 -- /bin/sh
/ # echo $NGINX_SERVER_PORT
80
/ # echo $NGINX_SERVER_NAME
myapp.magedu.com
```

修改端口，可以发现使用环境变化注入pod中的端口不会根据配置的更改而变化

```
[root@k8s-master volumes]#  kubectl edit cm nginx-config
configmap/nginx-config edited
/ # echo $NGINX_SERVER_PORT
80
```
2. 存储卷方式挂载configmap：Volume 形式的 ConfigMap 也支持动态更新

```
[root@k8s-master configmap ~]# vim pod-configmap-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-2
  namespace: default
  labels: 
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80 
    volumeMounts:
    - name: nginxconf
      mountPath: /etc/nginx/config.d/
      readOnly: true
  volumes:
  - name: nginxconf
    configMap:
      name: nginx-config
[root@k8s-master configmap ~]# kubectl apply -f pod-configmap-2.yaml
pod/pod-cm-2 created
[root@k8s-master configmap ~]# kubectl get pods
[root@k8s-master configmap ~]# kubectl exec -it pod-cm-2 -- /bin/sh
/ # cd /etc/nginx/config.d
/ # cat nginx_port
80
/ # cat server_name 
myapp.magedu.com

[root@k8s-master configmap ~]# kubectl edit cm nginx-config  #修改端口，再在容器中查看端口是否变化。
apiVersion: v1
data:
  nginx_port: "800"
  ......
  
/ # cat nginx_port
800
[root@k8s-master configmap ~]# kubectl delete -f pod-configmap2.yaml
```
3. 以nginx-www配置nginx

```
[root@k8s-master configmap ~]# vim pod-configmap3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-cm-3
  namespace: default
  labels: 
    app: myapp
    tier: frontend
  annotations:
    magedu.com/created-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80 
    volumeMounts:
    - name: nginxconf
      mountPath: /etc/nginx/conf.d/
      readOnly: true
  volumes:
  - name: nginxconf
    configMap:
      name: nginx-www
[root@k8s-master configmap ~]# kubectl apply -f pod-configmap3.yaml
pod/pod-cm-3 created
[root@k8s-master configmap ~]# kubectl get pods
[root@k8s-master configmap]# kubectl exec -it pod-cm-3 -- /bin/sh
/ # cd /etc/nginx/conf.d/
/etc/nginx/conf.d # ls
www.conf
/etc/nginx/conf.d # cat www.conf 
server {
    server_name myapp.magedu.com;
    listen 80;
    root /data/web/html;
}
```

至此，K8S的存储卷到此结束！！！

## 参考资料

> - []()
> - []()
