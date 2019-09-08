---
title: pv
toc: true
date: 2019-09-05 20:02:35
tags:
categories:
---



## 存储卷概念

为了保证数据的持久性，必须保证数据在外部存储。

在`docker`容器中，为了实现数据的持久性存储，在宿主机和容器内做映射，可以保证在容器的生命周期结束，数据依旧可以实现持久性存储。但是在`k8s`中，由于`pod`分布在各个不同的节点之上，并不能实现不同节点之间持久性数据的共享，并且，在节点故障时，可能会导致数据的永久性丢失。为此，`k8s`就引入了外部存储卷的功能。

k8s的存储卷类型，可通过下面命令查看：

```
kubectl explain pod.spec.volumes
```



- emptyDir: 临时目录，Pod删除，数据也会被清除，用于数据的临时存储
- hostPath: 宿主机目录映射
- 本地的SAN(iSCSI,FC)、NAS(nfs,cifs,http)存储
- 分布式存储: glusterfs，rbd，cephfs
- 云存储: EBS，Azure Disk





## emptyDir存储卷示例

一个emptyDir 第一次创建是在一个pod被指定到具体node的时候，并且会一直存在在pod的生命周期当中，正如它的名字一样，它初始化是一个空的目录，pod中的容器都可以读写这个目录，这个目录可以被挂在到各个容器相同或者不相同的的路径下。当一个pod因为任何原因被移除的时候，这些数据会被永久删除。注意：一个容器崩溃了不会导致数据的丢失，因为容器的崩溃并不移除pod.

默认的，emptyDir 磁盘会存储在主机所使用的媒介上，可能是SSD，或者网络硬盘，这主要取决于你的环境。当然，我们也可以将emptyDir.medium的值设置为Memory来告诉Kubernetes 来挂在一个基于内存的目录tmpfs，因为tmpfs速度会比硬盘块度了，但是，当主机重启的时候所有的数据都会丢失。

```
#查看emptyDir存储定义
kubectl explain pods.spec.volumes.emptyDir  

#查看容器挂载方式
kubectl explain pods.spec.containers.volumeMounts    
```

- 清单定义

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    magedu.com/create-by:"cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    volumeMounts:    #在容器内定义挂载存储名称和挂载路径
    - name: html
      mountPath: /usr/share/nginx/html/
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: html
      mountPath: /data/    #在容器内定义挂载存储名称和挂载路径
    command: ['/bin/sh','-c','while true;do echo $(date) >> /data/index.html;sleep 2;done']
  volumes:  #定义存储卷
  - name: html    #定义存储卷名称  
    emptyDir: {}  #定义存储卷类型
```

在上面，我们定义了2个容器，其中一个容器是输入日期到index.html中，然后验证访问nginx的html是否可以获取日期。以验证两个容器之间挂载的emptyDir实现共享。如下访问验证:

```
## 查看pod的ip
kubectl get pods -o wide

## 访问验证
curl 10.244.2.34  
```

## hostPath存储卷示例

hostPath宿主机路径，就是把pod所在的宿主机之上的脱离pod中的容器名称空间的之外的宿主机的文件系统的某一目录和pod建立关联关系，在pod删除时，存储数据不会丢失。

hostPath可以实现持久存储，但是在node节点故障时，也会导致数据的丢失

```
## 查看hostPath存储类型定义
kubectl explain pods.spec.volumes.hostPath  

path字段： 指定宿主机的路径
type字段：
        DirectoryOrCreate  宿主机上不存在创建此目录  
        Directory 必须存在挂载目录  
        FileOrCreate 宿主机上不存在挂载文件就创建  
        File 必须存在文件
```

1. 清单定义hostpath-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-hostpath
  namespace: default
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
  volumes:
    - name: html
      hostPath:
        path: /data/pod/volume1
        type: DirectoryOrCreate
```

2. 在node节点上创建挂载目录

```
## node01上
[root@k8s-node01 ~]# mkdir -p /data/pod/volume1
[root@k8s-node01 ~]# vim /data/pod/volume1/index.html
node01.yfming.com

## node02上
[root@k8s-node02 ~]# mkdir -p /data/pod/volume1
[root@k8s-node02 ~]# vim /data/pod/volume1/index.html
node02.yfming.com
```

3. 测试

```
## 应用配置
kubectl apply -f hostpath-pod.yaml

## 查看pod的ip
kubectl get pods -o wide

## 访问
curl 10.244.2.35

## 删除pod，再重建，验证是否依旧可以访问原来的内容
kubectl delete -f pod-hostpath-vol.yaml
```

## NFS共享存储卷示例

NFS使的我们可以挂在已经存在的共享到的我们的Pod中，和emptyDir不同的是，emptyDir会被删除当我们的Pod被删除的时候，但是NFS不会被删除，仅仅是解除挂在状态而已，这就意味着NFS能够允许我们提前对数据进行处理，而且这些数据可以在Pod之间相互传递.并且，NFS可以同时被多个pod挂在并进行读写

注意：必须先保证NFS服务器正常运行在我们进行挂在nfs的时候

1. 安装nfs

```
## 在stor01节点上安装nfs，并配置nfs服务
[root@stor01 ~]# yum install -y nfs-utils  ==》192.168.56.14
[root@stor01 ~]# mkdir /data/volumes -pv
[root@stor01 ~]# vim /etc/exports
/data/volumes 192.168.56.0/24(rw,no_root_squash)
[root@stor01 ~]# systemctl start nfs
[root@stor01 ~]# showmount -e
Export list for stor01:
/data/volumes 192.168.56.0/24

## 在node01和node02节点上安装nfs-utils，并测试挂载
[root@k8s-node01 ~]# yum install -y nfs-utils
[root@k8s-node02 ~]# yum install -y nfs-utils
[root@k8s-node02 ~]# mount -t nfs stor01:/data/volumes /mnt
[root@k8s-node02 ~]# mount
......
stor01:/data/volumes on /mnt type nfs4 (rw,relatime,vers=4.1,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=192.168.56.13,local_lock=none,addr=192.168.56.14)
[root@k8s-node02 ~]# umount /mnt/
```

2. 清单定义

```

```



## PVC和PV的概念

我们前面提到kubernetes提供那么多存储接口，但是首先kubernetes的各个Node节点能管理这些存储，但是各种存储参数也需要专业的存储工程师才能了解，由此我们的kubernetes管理变的更加复杂的。由此kubernetes提出了PV和PVC的概念，这样开发人员和使用者就不需要关注后端存储是什么，使用什么参数等问题。

pv和pvc是kubernetes抽象出来的一种存储资源。

PersistentVolume（PV）是集群中已由管理员配置的一段网络存储。 PV是诸如卷之类的卷插件，但是具有独立于使用PV的任何单个pod的生命周期。 
该API对象捕获存储的实现细节，即NFS，iSCSI或云提供商特定的存储系统。

PersistentVolumeClaim（PVC）是用户存储的请求。PVC的使用逻辑：在pod中定义一个存储卷（该存储卷类型为PVC），定义的时候直接指定大小，pvc必须与对应的pv建立关系，pvc会根据定义去pv申请，而pv是由存储空间创建出来的。

如下图：



![](pv/pvc-pv.png)







## 参考资料

> - []()
> - []()
