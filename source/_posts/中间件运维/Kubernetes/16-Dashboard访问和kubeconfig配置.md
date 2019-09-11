---
title: 13-Dashboard访问和kubeconfig配置
toc: true
date: 2019-09-03 10:06:55
tags:
categories:
---

在k8s中 dashboard可以有两种访问方式：kubeconfig（HTTPS）和token（http）



## token方式

```
## 创建名为def-ns-admin的ServiceAccount, 同时会自动生成名为def-ns-admin-token-xxx的secret
[root@master auth]# kubectl create serviceaccount def-ns-admin -n default
serviceaccount/def-ns-admin created

## 为def-ns-admin绑定集群角色admin,注意此处用的是rolebinding，只会对当前命名空间有权限
[root@master auth]# kubectl create rolebinding def-ns-admin --clusterrole=admin --serviceaccount=default:def-ns-admin
rolebinding.rbac.authorization.k8s.io/def-ns-admin created

## 查看token,将该token复制后，填入验证，要知道的是，该token认证仅可以查看default名称空间的内容
[root@master auth]# kubectl describe secret def-ns-admin-token-hsbpq
```



## kubeconfig配置文件

- 配置文件中用token认证

```
## 设置集群信息，到配置文件	
[root@master auth]# kubectl config set-cluster kubernetes --certificate-authority=/etc/kubernetes/pki/ca.crt --server="https://192.168.11.129:6443" --embed-certs=true --kubeconfig=/root/k8s/auth/def-ns-admin.conf

## 设置集群认证方式
## 认证的方式可以通过crt和key文件，也可以使用token进行配置，这里使用tonken
## 查看帮助
[root@master auth]# kubectl config set-credentials -h

## 查看token
[root@master auth]# kubectl describe secret def-ns-admin-token-hsbpq

## token是base64编码，此处需要进行解码操作
[root@master auth]# kubectl get secret def-ns-admin-token-hsbpq -o jsonpath={.data.token} | base64 -d

## 设置token, xxxx为上面解码出的token
[root@master auth]# kubectl config set-credentials def-ns-admin --token=xxxx --kubeconfig=/root/k8s/auth/def-ns-admin.conf

## 配置上下文
[root@master auth]# kubectl config set-context def-ns-admin@kubernetes --cluster=kubernetes --user=def-ns-admin --kubeconfig=/root/k8s/auth/def-ns-admin.conf 
Context "def-ns-admin@kubernetes" created.

## 设置当前上下文
[root@master auth]# kubectl config use-context def-ns-admin@kubernetes --kubeconfig=/root/k8s/auth/def-ns-admin.conf 
Switched to context "def-ns-admin@kubernetes".

## 查看kubeconfig配置
[root@master auth]# kubectl config view --kubeconfig=/root/k8s/auth/def-ns-admin.conf 
```

将def-ns-admin.conf文件发送到宿主机，浏览器访问时选择Kubeconfig认证，载入该配置文件，点击登陆，即可实现访问。

- 配置文件中证书认证

```
## 生成证书
[root@master auth]# (umask 077;openssl genrsa -out dashboard.key 2048)
Generating RSA private key, 2048 bit long modulus
..............................................+++
...................................................................................+++
e is 65537 (0x10001)

[root@master auth]# openssl req -new -key dashboard.key -out dashboard.csr -subj "/O=fuyi/CN=dashboard"

[root@master auth]# openssl x509 -req -in dashboard.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out dashboard.crt -days 365

## 生成配置文件
```

## 参考资料
> - []()
> - []()
