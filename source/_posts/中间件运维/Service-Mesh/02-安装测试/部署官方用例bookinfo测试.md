---
title: 部署官方用例bookinfo测试
toc: true
date: 2019-10-12 22:38:35
tags:
categories:
---





## bookinfo架构简介

官方提供的bookinfo示例，展示一本书的信息，用来演示istio特性。

该示例由如下四个微服务组成：

- productpage:  主服务，展示书的简介，并调用details和reviews服务获取书的详细信息和书评信息。
- details: 提供书的详细信息。
- reviews: 提供书评信息，并调用ratings服务获取书评等级。
- ratings: 提供书评等级信息。

reviews书评服务有三个版本：

- v1不调用ratings服务。
- v2调用ratings服务，并将每个等级显示为1到5个黑色星标。
- v3调用ratings服务，并将每个等级显示为1到5个红色星标。

该应用程序是多语言的，即微服务以不同的语言编写，架构图如下：

![](部署官方用例bookinfo测试/bookinfo-arch.svg)



## 部署bookinfo

要使用istio部署应用，无需对应用进行任何特殊改动，只需要在启用Istio的环境中配置和运行服务，istio会自动为每个服务旁边注入一个Envoy代理，最终部署如下：

![](部署官方用例bookinfo测试/bookinfo-arch1.svg)

该Envoy代理会拦截对相应服务的调用与响应，并通过Istio控制平面去配置路由、调用跟踪等策略实施控制。

```
## 1. 开启default命名空间的自动注入
kubectl label namespace default istio-injection=enabled

## 2. 部署bookinfo
kubectl apply -f /usr/local/istio/samples/bookinfo/platform/kube/bookinfo.yaml

## 3. 查看服务
kubectl get services
kubectl get pods

## 4. 访问测试
$ kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"

## 响应结果
<title>Simple Bookstore App</title>
```

现在服务在集群内部正常运行了，现在需要让集群外部可访问，Istio Gateway就是用这个目的。

```
## 5. 部署ingress gateway
kubectl apply -f /usr/local/istio/samples/bookinfo/networking/bookinfo-gateway.yaml

## 6. 查看状态

## 6.1 kubectl 查看
kubectl get gateway

## 6.2 istioctl 查看
istioctl get gateway

```



## 访问测试

```
## 1. 配置环境变量
## 1.1
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

## 1.2
NODE_NAME=$(kubectl get no | grep '<none>' | head -1 | awk '{print $1}')

## 1.3
NODE_IP=$(ping -c 1 $NODE_NAME | grep PING | awk '{print $3}' | tr -d '()')

## 1.4
export GATEWAY_URL=$NODE_IP:$INGRESS_PORT

## 1.5
echo $GATEWAY_URL

## 2. 命令行访问测试
curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage

## 3. 输出浏览器访问URL
echo "http://${GATEWAY_URL}/productpage"

## 4. 通过浏览器访问看到如下结果，多次刷新，reviews的3个版本会被依次调用，展示红星，黑星，无星，因为我们尚未使用Istio来控制版本路由。
```



![](部署官方用例bookinfo测试/bookinfo-result.png)

## 删除bookinfo

```
sh samples/bookinfo/platform/kube/cleanup.sh
```



## 附录





## 参考资料

> - []()
> - []()
