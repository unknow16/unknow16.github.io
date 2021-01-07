---
title: 12-存储卷：Secret
toc: true
date: 2019-08-31 01:07:51
tags:
categories:
---



在日常单机甚至集群状态下，我们需要对一个应用进行配置，只需要修改其配置文件即可。那么在容器中又该如何提供配置  信息呢？？？例如，为Nginx配置一个指定的server_name或worker进程数，为Tomcat的JVM配置其堆内存大小。传统的实践过程中通常有以下几种方式：

- 启动容器时，通过命令传递参数
- 将定义好的配置文件通过镜像文件进行写入
- 通过环境变量的方式传递配置数据
- 挂载Docker卷传送配置文件

而在Kubernetes系统之中也存在这样的组件，就是特殊的存储卷类型。其并不是提供pod存储空间，而是给管理员或用户提供从集群外部向Pod内部的应用注入配置信息的方式。这两种特殊类型的存储卷分别是：configMap和secret。



- Secret：用于向Pod传递敏感信息，比如密码，私钥，证书文件等，这些信息如果在容器中定义容易泄露，Secret资源可以让用户将这些信息存储在集群中，然后通过Pod进行挂载，实现敏感数据和系统解耦的效果。
- ConfigMap：主要用于向Pod注入非敏感数据，使用时，用户将数据直接存储在ConfigMap对象当中，然后Pod通过使用ConfigMap卷进行引用，实现容器的配置文件集中定义和管理。

```
[root@k8s-master volumes]# kubectl explain pods.spec.volumes
......
   configMap    <Object>
     ConfigMap represents a configMap that should populate this volume

   secret   <Object>
     Secret represents a secret that should populate this volume. More info:
     https://kubernetes.io/docs/concepts/storage/volumes#secret
```



## Secret简介

Secret对象存储数据的方式是以键值方式存储数据，在Pod资源进行调用Secret的方式是通过环境变量或者存储卷的方式进行访问数据，解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod   Spec中。另外，Secret对象的数据存储和打印格式为Base64编码的字符串，因此用户在创建Secret对象时，也需要提供该类型的编码格式的数据。在容器中以环境变量或存储卷的方式访问时，会自动解码为明文格式。需要注意的是，如果是在Master节点上，Secret对象以非加密的格式存储在etcd中，所以需要对etcd的管理和权限进行严格控制。

## Secret的4种类型

- Service Account ：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的/run/secrets/kubernetes.io/serviceaccount目录中；
- Opaque ：base64编码格式的Secret，用来存储密码、密钥、信息、证书等，类型标识符为generic；
- kubernetes.io/dockerconfigjson ：用来存储私有docker registry的认证信息，类型标识为docker-registry。
- kubernetes.io/tls：用于为SSL通信模式存储证书和私钥文件，命令式创建类型标识为tls。

## 创建 Secret的2种方式

**命令式创建**

1. 通过 --from-literal：

```
[root@k8s-master ~]# kubectl create secret -h
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

## 每个 --from-literal 对应一个信息条目。
[root@k8s-master ~]# kubectl create secret generic mysecret --from-literal=username=admin --from-literal=password=123456
secret/mysecret created
[root@k8s-master ~]# kubectl get secret
NAME                    TYPE                                  DATA      AGE
mysecret                Opaque                                2         6s
```

2. 通过 --from-file：每个文件内容对应一个信息条目

```
[root@k8s-master ~]# echo -n admin > ./username
[root@k8s-master ~]# echo -n 123456 > ./password
[root@k8s-master ~]# kubectl create secret generic mysecret --from-file=./username --from-file=./password 
secret/mysecret created
[root@k8s-master ~]# kubectl get secret
NAME                    TYPE                                  DATA      AGE
mysecret                Opaque                                2         6s
```

3. 通过 --from-env-file：文件 env.txt 中每行 Key=Value 对应一个信息条目

```
[root@k8s-master ~]# cat << EOF > env.txt
> username=admin
> password=123456
> EOF
[root@k8s-master ~]# kubectl create secret generic mysecret --from-env-file=env.txt 
secret/mysecret created
[root@k8s-master ~]# kubectl get secret
NAME                    TYPE                                  DATA      AGE
mysecret                Opaque                                2         10s
```

**清单式创建**：通过 YAML 配置文件：

```
#事先完成敏感数据的Base64编码
[root@k8s-master ~]# echo -n admin |base64
YWRtaW4=
[root@k8s-master ~]# echo -n 123456 |base64
MTIzNDU2

[root@k8s-master ~]# vim secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  username: YWRtaW4=
  password: MTIzNDU2
[root@k8s-master ~]# kubectl apply -f secret.yaml 
secret/mysecret created
[root@k8s-master ~]# kubectl get secret  #查看存在的 secret，显示有2条数据
NAME                    TYPE                                  DATA      AGE
mysecret                Opaque                                2         8s
[root@k8s-master ~]# kubectl describe secret mysecret  #查看数据的 Key
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  
Type:         Opaque

Data
====
username:  5 bytes
password:  6 bytes
[root@k8s-master ~]# kubectl edit secret mysecret  #查看具体的value，可以使用该命令
apiVersion: v1
data:
  password: MTIzNDU2
  username: YWRtaW4=
kind: Secret
metadata:
......
[root@k8s-master ~]# echo -n MTIzNDU2 |base64 --decode  #通过 base64 将 Value 反编码：
123456
[root@k8s-master ~]# echo -n YWRtaW4= |base64 --decode
admin
```

## 如何使用Secret？

Pod 可以通过 Volume 或者环境变量的方式使用 Secret



1. 可以看到，Kubernetes 会在指定的路径 /etc/foo 下为每条敏感数据创建一个文件，文件名就是数据条目的 Key，这里是 /etc/foo/username 和 /etc/foo/password，Value 则以明文存放在文件中。

```
[root@k8s-master volumes]# vim pod-secret-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
spec:
  containers:
  - name: pod-secret
    image: busybox
    args:
      - /bin/sh
      - -c
      - sleep 10;touch /tmp/healthy;sleep 30000
    volumeMounts:   #将 foo mount 到容器路径 /etc/foo，可指定读写权限为 readOnly。
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:    #定义 volume foo，来源为 secret mysecret。
  - name: foo
    secret:
      secretName: mysecret
[root@k8s-master volumes]# kubectl apply -f pod-secret-demo.yaml 
pod/pod-secret created
[root@k8s-master volumes]# kubectl get pods
pod-secret                           1/1       Running   0          1m
[root@k8s-master volumes]# kubectl exec -it pod-secret sh
/ # ls /etc/foo/
password  username
/ # cat /etc/foo/username 
admin/ # 
/ # cat /etc/foo/password 
123456/ # 
```



2. 也可以自定义存放数据的文件名，比如将配置文件改为：

```
[root@k8s-master volumes]# cat pod-secret-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
spec:
  containers:
  - name: pod-secret
    image: busybox
    args:
      - /bin/sh
      - -c
      - sleep 10;touch /tmp/healthy;sleep 30000
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:    #自定义存放数据的文件名
      - key: username
        path: my-secret/my-username
      - key: password
        path: my-secret/my-password
[root@k8s-master volumes]# kubectl delete pods pod-secret
pod "pod-secret" deleted
[root@k8s-master volumes]# kubectl apply -f pod-secret-demo.yaml 
pod/pod-secret created
[root@k8s-master volumes]# kubectl exec -it pod-secret sh
/ # cat /etc/foo/my-secret/my-username 
admin
/ # cat /etc/foo/my-secret/my-password 
123456
```

这时数据将分别存放在 /etc/foo/my-secret/my-username 和 /etc/foo/my-secret/my-password 中。

以 Volume 方式使用的 Secret 支持动态更新：Secret 更新后，容器中的数据也会更新。



将 password 更新为 abcdef，base64 编码为 YWJjZGVm

```
[root@k8s-master ~]# vim secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  username: YWRtaW4=
  password: YWJjZGVm
[root@k8s-master ~]# kubectl apply -f secret.yaml 
secret/mysecret configured
/ # cat /etc/foo/my-secret/my-password 
abcdef
```

3. 通过 Volume 使用 Secret，容器必须从文件读取数据，会稍显麻烦，Kubernetes 还支持通过环境变量使用 Secret。

```
[root@k8s-master volumes]# cp pod-secret-demo.yaml pod-secret-env-demo.yaml
[root@k8s-master volumes]# vim pod-secret-env-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-env
spec:
  containers:
  - name: pod-secret-env
    image: busybox
    args:
      - /bin/sh
      - -c
      - sleep 10;touch /tmp/healthy;sleep 30000
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
[root@k8s-master volumes]# kubectl apply -f pod-secret-env-demo.yaml 
pod/pod-secret-env created

[root@k8s-master volumes]# kubectl exec -it pod-secret-env sh
/ # echo $SECRET_USERNAME
admin
/ # echo $SECRET_PASSWORD
abcdef
```

通过环境变量 SECRET_USERNAME 和 SECRET_PASSWORD 成功读取到 Secret 的数据。需要注意的是，环境变量读取 Secret 很方便，但无法支撑 Secret 动态更新。Secret 可以为 Pod 提供密码、Token、私钥等敏感数据；对于一些非敏感数据，比如应用的配置信息，则可以用 ConfigMap。

## 参考资料
> - []()
> - []()
