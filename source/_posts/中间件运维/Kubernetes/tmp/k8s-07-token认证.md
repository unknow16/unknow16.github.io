---
title: k8s-07-token认证
date: 2018-08-05 19:26:32
tags: Kubernetes
---

当我们有非常多的node节点时，手动为每个node节点配置TLS认证比较麻烦，这时就可以用到token的认证方式，客户端的token信息与服务端预先定义的token匹配认证通过后，自动为node颁发证书。当然引导token是一种机制，可以用到各种场景中。

根据服务端存储token的方式不同，token认证分为两种：

- 静态token
- 引导token / bootstrap-token

前者token是存放在服务端的一个csv文件中，而后者token以Secrets资源的方式存储在kube-system命名空间中，在这个命名空间中的token可以被动态的管理和创建。Controller Manager有一个管理中心，如果token过期了就会删除。

## 静态token认证

token是存放在服务端的一个csv文件中，token唯一标识请求者，只要apiserver存在该token，则认为认证通过，但是如果需要新增Token，则需要重启kube-apiserver组件，实际效果不是很好。


令牌文件是一个至少包含3列的csv文件： token, user name, user uid，后跟可选的组名。注意，如果您有多个组，则列必须是双引号，例如：


```
token,user,uid,"group1,group2,group3"
```
如下示例：
```
8afdf3c4eb7c74018452423c29433609,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
```


当通过客户端使用 bearer token 认证时，kube-apiserver需要一个值为Bearer THETOKEN的授权头。bearer token必须是可以放在HTTP请求头中且值不需要转码和引用的一个字符串。

例如：如果bearer token是31ada4fd-adec-460c-809a-9e56ceb75269，它将会在HTTP头中按下面的方式呈现：


```
Authorization: Bearer 31ada4fd-adec-460c-809a-9e56ceb75269
```
#### 启用方式
当给kube-apiserver进程加如下启动参数时，kube-apiserver会从指定文件中读取 bearer tokens。目前，tokens持续无限期。

```
--token-auth-file=/etc/kubernetes/ca/kubernetes/token.csv
```

## 引导Token
这种token以Secrets的方式存储在kube-system命名空间中，在这个命名空间token可以被动态的管理和创建。Controller Manager有一个管理中心，如果token过期了就会删除。

创建的token证书满足[a-z0-9]{6}.[a-z0-9]{16}格式，Token的第一部分是一个Token ID，第二部分是token的秘钥。你需要在http协议头中加上类似的信息：


```
Authorization: Bearer 781292.db7bc3a58fc5f07e
```

#### 启用方式
需要在kube-apiserver启动时添加如下参数

```
enable-bootstrap-token-auth
```
同时必须在Controller Manager中开启管理中心的设置
```
--controllers=*,tokencleaner
```

在使用kubeadm部署Kubernetes时，kubeadm会自动创建默认token，可通过kubeadm token list命令查询。


## token认证过程

此处以静态token文件为例说明过程

#### 1. 服务端生成token认证文件

- 生成随机token
```
# 生成随机token
$ head -c 16 /dev/urandom | od -An -t x | tr -d ' '
8afdf3c4eb7c74018452423c29433609
```
- 按照固定格式写入token.csv，注意替换token内容
```
# 按照固定格式写入token.csv，注意替换token内容
$ echo "8afdf3c4eb7c74018452423c29433609,kubelet-bootstrap,10001,\"system:kubelet-bootstrap\"" > /etc/kubernetes/ca/kubernetes/token.csv
```
kube-apiserver要求该用户是具有一个特定的角色：system:node-bootstrapper。

所以需要先将静态token文件中的 kubelet-bootstrap 用户赋予这个特定角色，然后 kubelet 才有权限发起创建认证请求。


```
在主节点执行下面命令

#可以通过下面命令查询clusterrole列表
$ kubectl -n kube-system get clusterrole

#可以回顾一下token文件的内容
$ cat /etc/kubernetes/ca/kubernetes/token.csv
8afdf3c4eb7c74018452423c29433609,kubelet-bootstrap,10001,"system:kubelet-bootstrap"

#创建角色绑定（将用户kubelet-bootstrap与角色system:node-bootstrapper绑定）
$ kubectl create clusterrolebinding kubelet-bootstrap \
         --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
```
#### 2. kube-apiserver启用静态token 
kube-apiserver配置文件中，加如下启动参数：
```
--token-auth-file=/etc/kubernetes/ca/kubernetes/token.csv
```
如果是引导token认证，还需对kube-controller-manager配置如下启动参数：
```
--controllers=*,tokencleaner
```


#### 3. 工作节点创建bootstrap.kubeconfig
客户端的kubelet访问kube-apiserver发起请求认证时，需要告诉它用户名和token。

这个配置保存了像用户名，token等重要的认证信息，这个文件可以借助kubectl命令生成：（也可以自己写配置）

```
#设置集群参数(注意替换ip)
$ kubectl config set-cluster kubernetes \
        --certificate-authority=/etc/kubernetes/ca/ca.pem \
        --embed-certs=true \
        --server=https://192.168.11.128:6443 \
        --kubeconfig=bootstrap.kubeconfig
#设置客户端认证参数(注意替换token)
$ kubectl config set-credentials kubelet-bootstrap \
        --token=8afdf3c4eb7c74018452423c29433609 \
        --kubeconfig=bootstrap.kubeconfig
#设置上下文
$ kubectl config set-context default \
        --cluster=kubernetes \
        --user=kubelet-bootstrap \
        --kubeconfig=bootstrap.kubeconfig
#选择上下文
$ kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
#将刚生成的文件移动到合适的位置
$ mv bootstrap.kubeconfig /etc/kubernetes/
```

#### 4. kubelet配置认证信息
kubelet.service配置文件中，加如下启动参数：
```
--experimental-bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig 
```

