---
title: 通过Kiali可视化Mesh
toc: true
date: 2019-11-16 12:26:29
tags:
categories:
---



安装Kiali附件





本篇安装Kiali附件，基于图形界面可视化mesh。



## 安装

1. 创建一个Secret

- 定义之后用来登录Kiali的用户名和密码

```
$ KIALI_USERNAME=$(read -p 'Kiali Username: ' uval && echo -n $uval | base64)

$ KIALI_PASSPHRASE=$(read -sp 'Kiali Passphrase: ' pval && echo -n $pval | base64)
```

- 运行下面命令创建secret

```
$ NAMESPACE=istio-system

$ kubectl create namespace $NAMESPACE
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: $NAMESPACE
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

- 创建secret

```
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
```



2. 通过helm生成部署文件

```
$ helm template --set kiali.enabled=true install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml

$ kubectl apply -f $HOME/istio.yaml
```

如果你已经安装了jaeger和Grafana,想集成，则使用下面命

```
$ helm template \
    --set kiali.enabled=true \
    --set "kiali.dashboard.jaegerURL=http://$(kubectl get svc tracing -o jsonpath='{.spec.clusterIP}'):80" \
    --set "kiali.dashboard.grafanaURL=http://$(kubectl get svc grafana -o jsonpath='{.spec.clusterIP}'):3000" \
    install/kubernetes/helm/istio \
    --name istio --namespace istio-system > $HOME/istio.yaml

$ kubectl apply -f $HOME/istio.yaml
```



## 访问UI

1. 查看kiali服务

```
$ kubectl -n istio-system get svc kiali
```

2. 暴露外部访问kiali的UI，有两种方法

- 端口转发，需要iptables支持，如果防火墙关了是不行的

```
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```

访问地址：http://ip: 20001/kiali，然后输入登录账户密码，值是之前创建Secret时设置的

- 使用nodePort，自行修改



## 关于Kiali Public API

通过Kiali Public API，可以生成JSON格式数据，图表、健康、配置信息等指标。

-  `$KIALI_URL/api/namespaces/graph?namespaces=bookinfo&graphType=app`

如上是app图类型的JSON表示

Kiali Public API建立在Prometheus查询之上，并且取决于标准的Istio度量配置。它还会调用Kubernetes API以获取有关您的服务的其他详细信息。为了获得使用Kiali的最佳体验，请在应用程序组件上使用元数据标签app和version。作为模板，Bookinfo示例应用程序遵循此约定。



## 清除环境

```
$ kubectl delete all,secrets,sa,configmaps,deployments,ingresses,clusterroles,clusterrolebindings,virtualservices,destinationrules --selector=app=kiali -n istio-system
```



## 参考资料

> - []()
> - []()
