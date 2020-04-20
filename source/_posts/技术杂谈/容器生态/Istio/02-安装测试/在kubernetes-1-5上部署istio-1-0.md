---
title: 在kubernetes-1.5上部署istio-1.0
toc: true
date: 2019-10-12 20:16:46
tags:
categories:
---



`istio`是一个`service mesh`开源实现，由Google/IBM/Lyft共同开发。



## 下载并配置环境变量

```
## 1. 下载
wget https://github.com/istio/istio/releases/download/1.0.0/istio-1.0.0-linux.tar.gz

## 2. 解压
tar zxvf istio-1.0.0-linux.tar.gz

## 3. 移动安装包
mv istio-1.0.0 /usr/local/

## 4. 设置链接
ln -sv /usr/local/istio-1.0.0 /usr/local/istio

## 5. 写入环境变量
echo 'export PATH=/usr/local/istio/bin:$PATH' > /etc/profile.d/istio.sh

## 6. 使配置生效
source /etc/profile.d/istio.sh

## 7. 查看istioctl版本,测试环境变量
istioctl version
```



## 在k8s中部署istio

部署文件中需要修改下面两个地方

1. 部署文件中name是ingressgateway的Deployment改为DaemonSet类型方式。
2. 默认部署文件中name是ingressgateway的Service的type是LoadBalancer，如果环境不是云环境，需要修改为NodePort。

```
## 1. 进入目录
cd /usr/local/istio

## 2. 备份部署文件
cp install/kubernetes/istio-demo.yaml install/kubernetes/istio-demo.yaml.ori

## 3. 修改配置
vim install/kubernetes/istio-demo.yaml
```

istio-demo.yaml中需要修改的部分，即加了注释的地方

```
## 大约两千四百多行
...
apiVersion: v1
kind: Service
metadata:
  name: istio-ingressgateway
  namespace: istio-system
  annotations:
  labels:
    chart: gateways-1.0.0
    release: RELEASE-NAME
    heritage: Tiller
    app: istio-ingressgateway
    istio: ingressgateway
spec:
  # 默认是LoadBalancer，此处修改为NodePort
  type: NodePort
  selector:
    app: istio-ingressgateway
    istio: ingressgateway

...
## 大约在三千多行
apiVersion: extensions/v1beta1
# 默认是Deployment，此处改为DaemonSet部署方式
kind: DaemonSet
metadata:
  name: istio-ingressgateway
  namespace: istio-system
  labels:
    app: ingressgateway
    chart: gateways-1.0.0
    release: RELEASE-NAME
    heritage: Tiller
    app: istio-ingressgateway
    istio: ingressgateway
spec:
  # DaemonSet不支持replicas，此处注释掉
  # replicas: 1
  template:
    metadata:
      labels:
        app: istio-ingressgateway
        istio: ingressgateway
      annotations:
        sidecar.istio.io/inject: "false"
        scheduler.alpha.kubernetes.io/critical-pod: ""
    spec:
      serviceAccountName: istio-ingressgateway-service-account
      containers:
        - name: ingressgateway
          image: "gcr.io/istio-release/proxyv2:1.0.0"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              # 增加主机80端口映射
              hostPort: 80
            - containerPort: 443
              # 增加主机443端口映射
              hostPort: 443
...
```

```
## 4. 替换镜像地址
## 4.1 替换gcr.io相关镜像
sed -i 's@gcr.io/istio-release@docker.io/istio@g' install/kubernetes/istio-demo.yaml

## 4.2 替换quay.io相关镜像
sed -i 's@quay.io/coreos/hyperkube:v1.7.6_coreos.0@registry.cn-shanghai.aliyuncs.com/gcr-k8s/hyperkube:v1.7.6_coreos.0@g' install/kubernetes/istio-demo.yaml

## 5. 查看所有镜像地址
grep 'image:' install/kubernetes/istio-demo.yaml

## 5.1 在k8s集群node上提前下载上面的镜像
docker pull xxxx

## 6. 安装 CRDs
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system

## 6.1 查看crd
kubectl get crd

## 7. 部署istio，如遇istio-pilot不正常，解决方案看下文
kubectl apply -f install/kubernetes/istio-demo.yaml

## 8. 查看状态
kubectl get svc -n istio-system
kubectl get pods -n istio-system
```

如遇该问题：istio-pilot nodes are available: 3 Insufficient memory，此时可增加内存到3G或修改部署文件中的资源限制，修改部分如下：

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: istio-pilot
  namespace: istio-system
  # TODO: default template doesn't have this, which one is right ?
  labels:
    app: istio-pilot
    chart: pilot-1.0.0
    release: RELEASE-NAME
    heritage: Tiller
    istio: pilot
  annotations:
    checksum/config-volume: f8da08b6b8c170dde721efd680270b2901e750d4aa186ebb6c22bef5b78a43f9
spec:
  replicas: 1
  template:
    metadata:
      labels:
        istio: pilot
        app: pilot
      annotations:
        sidecar.istio.io/inject: "false"
        scheduler.alpha.kubernetes.io/critical-pod: ""
    spec:
      serviceAccountName: istio-pilot-service-account
      containers:
        - name: discovery
          image: "docker.io/istio/pilot:1.0.0"
          imagePullPolicy: IfNotPresent
          args:
          - "discovery"
          ports:
          - containerPort: 8080
          - containerPort: 15010
          readinessProbe:
            httpGet:
              path: /debug/endpointz
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 30
            timeoutSeconds: 5
          env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
          - name: PILOT_THROTTLE
            value: "500"
          - name: PILOT_CACHE_SQUASH
            value: "5"
          - name: PILOT_TRACE_SAMPLING
            value: "100"
          # 注释掉下面的资源请求
          #resources:
          #  requests:
          #    cpu: 500m
          #    memory: 2048Mi
          volumeMounts:
          - name: config-volume
            mountPath: /etc/istio/config
          - name: istio-certs
            mountPath: /etc/certs
            readOnly: true
```



## 配置双向TLS

有两种方式：宽松模式和严格模式

- permissive mutual TLS mode，宽松模式

所有服务均接受纯文本和双向TLS流量。除非配置为双向TLS，否则客户端不进行加密，而发送纯文本流量。有如下两种情况时适用：

1. 集群已存在应用
2. 使用Istio边车的服务需要能够与其他非Istio Kubernetes服务进行通信的应用程序

```
## 安装方法:应用istio安装包中install/kubernetes/istio-demo.yaml
$ kubectl apply -f install/kubernetes/istio-demo.yaml
```



- strict mutual TLS mode，严格模式

该模式强制客户端和服务端双方都要使用TLS。仅在所有工作负载均支持Istio的全新Kubernetes集群上使用此模式。所有新部署的工作负载都将安装Istio sidecar。

```
## 安装方法: 应用istio安装包中install/kubernetes/istio-demo-auth.yaml
$ kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```



## 校验安装

确保已部署以下Kubernetes服务，并确认除了jaeger-agent服务之外，它们都具有适当的CLUSTER-IP。

```
$  kubectl get svc -n istio-system
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                                                     AGE
grafana                    ClusterIP   10.1.171.102   <none>        3000/TCP                                                                                                    25d
istio-citadel              ClusterIP   10.1.1.159     <none>        8060/TCP,9093/TCP                                                                                           25d
istio-egressgateway        ClusterIP   10.1.202.206   <none>        80/TCP,443/TCP                                                                                              25d
istio-galley               ClusterIP   10.1.39.84     <none>        443/TCP,9093/TCP                                                                                            25d
istio-ingressgateway       NodePort    10.1.75.185    <none>        80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:30981/TCP,8060:32240/TCP,15030:30038/TCP,15031:32395/TCP   25d
istio-pilot                ClusterIP   10.1.169.136   <none>        15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                       25d
istio-policy               ClusterIP   10.1.229.111   <none>        9091/TCP,15004/TCP,9093/TCP                                                                                 25d
istio-sidecar-injector     ClusterIP   10.1.162.58    <none>        443/TCP                                                                                                     25d
istio-statsd-prom-bridge   ClusterIP   10.1.132.134   <none>        9102/TCP,9125/UDP                                                                                           25d
istio-telemetry            ClusterIP   10.1.193.85    <none>        9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                       25d
jaeger-agent               ClusterIP   None           <none>        5775/UDP,6831/UDP,6832/UDP                                                                                  25d
jaeger-collector           ClusterIP   10.1.155.149   <none>        14267/TCP,14268/TCP                                                                                         25d
jaeger-query               ClusterIP   10.1.207.226   <none>        16686/TCP                                                                                                   25d
prometheus                 ClusterIP   10.1.173.163   <none>        9090/TCP                                                                                                    25d
servicegraph               ClusterIP   10.1.191.61    <none>        8088/TCP                                                                                                    25d
tracing                    ClusterIP   10.1.104.173   <none>        80/TCP                                                                                                      25d
zipkin                     ClusterIP   10.1.21.177    <none>        9411/TCP                                                                                                    25d
```

确保已部署相应的Kubernetes Pod并具有运行状态：

```
$  kubectl get pods -n istio-system
NAME                                       READY   STATUS    RESTARTS   AGE
grafana-5f5888bf65-nvjbg                   1/1     Running   9          25d
istio-citadel-86c67595f6-zxxwz             1/1     Running   9          25d
istio-egressgateway-79b744756d-bchtd       1/1     Running   10         25d
istio-galley-f79d5df65-hrl2w               1/1     Running   25         24d
istio-ingressgateway-nmf4k                 1/1     Running   12         25d
istio-ingressgateway-rxksz                 1/1     Running   10         25d
istio-pilot-6b47746b85-7x5tw               2/2     Running   18         25d
istio-policy-95d4b5cf4-wrnm9               2/2     Running   23         25d
istio-sidecar-injector-66f4dd49bb-qjrf5    1/1     Running   7          25d
istio-statsd-prom-bridge-f575fdb46-lnrc5   1/1     Running   9          25d
istio-telemetry-8595d7f88c-qtqxq           2/2     Running   20         25d
istio-tracing-6c7b5d9498-k4rf9             1/1     Running   16         25d
prometheus-74f9669cd8-wsd6d                1/1     Running   13         25d
servicegraph-57b574467c-nghmx              1/1     Running   24         25d
```





## 启用自动注入sidecar

`istio-1.0.0` 默认已经开启了自动注入功能以及其他日志监控和追踪的相关组件如istio-tracing、istio-telemetry、grafana、prometheus、servicegraph。



> 准备工作

1. k8s-1.9及之后的版本才能使用自动注入功能，可通过如下命令查看是否支持

```
kubectl api-versions | grep admissionregistration
```

2. 除了要满足以上条件外还需要检查kube-apiserver启动的参数

- k8s-1.9 版本要确保  --admission-control 里有 MutatingAdmissionWebhook,ValidatingAdmissionWebhook
- k8s-1.9 之后的版本要确保 --enable-admission-plugins 里有MutatingAdmissionWebhook,ValidatingAdmissionWebhook

使用kubeadm安装的k8s集群，使用如下命令查看kube-apiserver

```
## 获取apiserver容器id
docker ps | grep apiserver

## 查看apiserver容器详细JSON信息中的Args数组中
docker inspect 8a743b193118
```

如果admission-control 或enable-admission-plugins无MutatingAdmissionWebhook,ValidatingAdmissionWebhook，则可通过修改配置文件进行增加，配置文件路径： /etc/kubernetes/manifests/kube-apiserver.yaml

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.11.129
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    
    ## 此处新增MutatingAdmissionWebhook,ValidatingAdmissionWebhook
    - --enable-admission-plugins=NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.1.0.0/16
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.11.129
        path: /healthz
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-apiserver
    resources:
      requests:
        cpu: 250m
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
status: {}
```

修改后，你会发现kube-apiserver会被自动重启，这是kubelet的功劳。kubelet在启动时监听/etc/kubernetes/manifests目录下的文件变化并做适当处理。 

> 测试自动注入

```
## 1. 自动注入前创建的pod中只有一个指定的sleep应用容器
kubectl apply -f samples/sleep/sleep.yaml 
kubectl get pod
kubectl describe pod sleep-xxxx

## 2. 开启 default 命名空间自动注入，其实就是设置标签istio-injection=enabled
kubectl label namespace default istio-injection=enabled
kubectl get namespace -L istio-injection

## 3. 删除创建的pod，等待重建
kubectl delete pod $(kubectl get pod | grep sleep | cut -d ' ' -f 1)

## 4. 查看新建pod有istio-proxy容器,即注入成功
kubectl get pod
kubectl describe pod $(kubectl get pod | grep sleep | cut -d ' ' -f 1)
```

> 手动注入

如果一个namespace没有istio-injection标签，你能使用istioctl kube-inject去注入sidecar-proxy容器

```
istioctl kube-inject -f <your-app-spec>.yaml | kubectl apply -f -
```



## 卸载istio

```
kubectl delete -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system

kubectl delete -f install/kubernetes/istio-demo.yaml

kubectl delete -f install/kubernetes/istio-demo-auth.yaml
```



## 参考资料

> - []()
> - []()
