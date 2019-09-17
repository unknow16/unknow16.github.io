---
title: 02-用kubeadm搭建一个v1-15的Kubernetes集群
toc: true
date: 2019-08-29 10:09:00
tags:
categories:
---

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。



## 安装要求
在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

## 所有节点准备环境

每台机器都需要执行

```
同步时间：
$ yum -y install ntp ntpdate
$ ntpdate cn.pool.ntp.org

关闭防火墙：
$ systemctl stop firewalld
$ systemctl disable firewalld

关闭selinux：
$ sed -i 's/enforcing/disabled/' /etc/selinux/config 
$ setenforce 0

关闭swap：
$ swapoff -a $ 临时
$ vim /etc/fstab $ 永久

添加主机名与IP对应关系，记得设置主机名：
$ cat /etc/hosts
192.168.31.61 k8s-master
192.168.31.62 k8s-node1
192.168.31.63 k8s-node2

将桥接的IPv4流量传递到iptables的链：
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system
```

##  所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

1. 安装Docker

```
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
Docker version 18.06.1-ce, build e68fc7a
```

2. 添加阿里云YUM软件源

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

3. 安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号1.15部署：

```
$ yum install -y kubelet-1.15.0 kubeadm-1.15.0 kubectl-1.15.0
$ systemctl enable kubelet
```

## Master节点安装

1. 初始化

- 注意修改下面ip
- 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址
- 留意以下命令执行完后的打印的kubeadm join xxx，后面从节点加入要使用

```
$ kubeadm init \
--apiserver-advertise-address=192.168.31.61 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.15.0 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16
```

2. 配置kubectl工具

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. 安装Pod网络插件（CNI）

- 确保能够访问到quay.io这个registery
- 如果不能，修改kube-flannel.yml成这个镜像地址：lizhenliang/flannel:v0.11.0-amd64

```
$ wget https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
$ kubectl apply -f kube-flannel.yml
```

## Node节点安装

执行在kubeadm init输出的kubeadm join命令，类似下面这样：

```
$ kubeadm join 192.168.31.61:6443 --token esce21.q6hetwm8si29qxwn \
--discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

## 测试集群

1. 部署nginx服务

```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get svc
```

会有类似如下输出，注意NodePort端口： 31439

```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.1.0.1     <none>        443/TCP        17h
nginx        NodePort    10.1.13.39   <none>        80:31439/TCP   17h
```

2. 访问http://NodeIP:31439

## 部署 Dashboard

1. 下载部署文件

```
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

2. 修改镜像地址

- 默认镜像国内无法访问，修改镜像地址为： lizhenliang/kubernetes-dashboard-amd64:v1.10.1
- 默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部
- 如下附需修改部分，共3处

```
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: lizhenliang/kubernetes-dashboard-amd64:v1.10.1 # 修改处
        ports:
        - containerPort: 8443
          protocol: TCP
        args:

。。。。。。。。。。。。。。。。。。。。。。。

# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort  # 新增
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001 # 新增
  selector:
    k8s-app: kubernetes-dashboard
```

3. 部署dashboard

```
$ kubectl apply -f kubernetes-dashboard.yaml
```

4. 创建service account并绑定默认cluster-admin管理员集群角色

- 最后会打印出token值，注意保留，后面登录要用

```
$ kubectl create serviceaccount dashboard-admin -n kube-system
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
$ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

5. 访问https://NodeIP:30001

- 注意是https前缀
- 用Chrome不行
- 建议用FireFox，添加https到例外

## 参考资料

> - []()
> - []()
