---
title: 15-暴露服务：Ingress和Ingress-Controller-Nginx
toc: true
date: 2019-09-01 16:20:53
tags:
categories:
---



## 目录

1. Ingress-Controller-Nginx运行原理
2. 安装Ingress-Controller-Nginx
3. 配置Ingress基于虚拟主机的转发规则
4. 配置Ingress基于URL的转发规则
5. 配置Ingress的TLS支持



## Ingress-Controller-Nginx运行原理

我们知道Service可以通过NodePort方式暴露服务，但是服务一旦多起来，NodePort 在每个节点上开启的端口会及其庞大，而且难以维护，这时，我们可以能否使用一个Nginx直接对内进行转发呢？当然可以。

简单的实现就是使用 Service 的NodePort在每个 Node 上监听 80，然后写好规则，因为 Nginx 外面绑定了宿主机 80 端口，本身又在集群内，那么向后直接转发到相应 Service IP 就行了。

从上面的方法，采用 Nginx-Pod 似乎已经解决了问题，但是其实这里面有一个很大缺陷：当每次有新服务加入又该如何修改 Nginx  配置呢？？我们知道使用Nginx可以通过虚拟主机域名进行区分不同的服务，而每个服务通过upstream进行定义不同的负载均衡池，再加上location进行负载均衡的反向代理，在日常使用中只需要修改nginx.conf即可实现，那在K8S中又该如何实现这种方式的调度呢？？？

假设后端的服务初始服务只有ecshop，后面增加了bbs和member服务，那么又该如何将这2个服务加入到Nginx-Pod进行调度呢？总不能每次手动改或者Rolling  Update 前端 Nginx Pod 吧！！此时 Ingress 出现了，如果不算上面的Nginx，Ingress  包含两大组件：Ingress Controller 和 Ingress。

Ingress 简单的理解就是你原来需要改 Nginx 配置，然后配置各种域名对应哪个 Service，现在把这个动作抽象出来，变成一个  Ingress 对象，你可以用 yaml 创建，每次不要去改 Nginx 了，直接改 yaml 然后创建/更新就行了；那么问题来了：”Nginx  该怎么处理？”

Ingress Controller 这东西就是解决 “Nginx 的处理方式” 的；Ingress Controoler 通过与  Kubernetes API 交互，动态的去感知集群中 Ingress 规则变化，然后读取他，按照他自己模板生成一段 Nginx 配置，再写到  Nginx Pod 里，最后 reload 一下，就ok了。

实际上Ingress也是Kubernetes API的标准资源类型之一，它其实就是一组基于DNS名称（host）或URL路径把请求转发到指定的Service资源的规则。用于将集群外部的请求流量转发到集群内部完成的服务发布。

我们需要明白的是，Ingress资源自身不能进行“流量穿透”，仅仅是一组规则的集合，这些集合规则还需要其他功能的辅助，比如监听某套接字，然后根据这些规则的匹配进行路由转发，这些能够为Ingress资源监听套接字并将流量转发的组件就是Ingress Controller。

Ingress 控制器不同于Deployment 控制器的是，Ingress控制器不直接运行为kube-controller-manager的一部分，它仅仅是Kubernetes集群的一个附件，类似于CoreDNS，需要在集群上单独部署。

目前主流的Ingress 控制器的选型实现是nginx和traefik。



## 安装Ingress-Controller-Nginx

官网：https://kubernetes.github.io/ingress-nginx/

1. ingress-controller-nginx-deploy.yaml

```
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: scolib/nginx-ingress-controller:0.25.1
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10

---
```

2. ingress-controller-nginx-svc.yaml

如下使用NodePort分别监听了http和https的30080、30443端口

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 30443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
```

## 配置Ingress基于虚拟主机的转发规则

实验目标：

- 访问tomcat.yfming.com，请求被转发到tomcat服务

- 访问nginx.yfming.com，请求被转发到nginx服务



部署tomcat和nginx服务: 

- tomcat-deploy-svc.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: tomcat
  namespace: default
spec:
  selector:
    app: tomcat
    release: canary
  ports:
  - name: http
    targetPort: 8080
    port: 8080
  - name: ajp
    targetPort: 8009
    port: 8009
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploy
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
      release: canary
  template:
    metadata:
      labels:
        app: tomcat
        release: canary
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5.34-jre8-alpine   
        ports:
        - name: http
          containerPort: 8080
          name: ajp
          containerPort: 8009
```

- nginx-deploy-svc.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  selector:
    app: nginx
    release: canary
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      release: canary
  template:
    metadata:
      labels:
        app: nginx
        release: canary
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3-alpine
        ports:
        - name: http
          containerPort: 80
```



配置Ingress规则

- tomcat-ingress.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: tomcat.yfming.com
    http:
      paths:
      - path:
        backend:
          serviceName: tomcat
          servicePort: 8080
```

- nginx-ingress.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: nginx.yfming.com
    http:
      paths:
      - path:
        backend:
          serviceName: nginx
          servicePort: 80
```

本地添加hosts映射，然后测试

- 访问tomcat.yfming.com:30080，正常响应tomcat首页

- 访问nginx.yfming.com:30080，正常响应nginx首页

可以进入ingress-controller的pod中查看ingress规则映射到nginx.conf的结果

```
kubectl exec -n ingress-nginx -it nginx-ingress-controller-6bd7c597cb-6pchv -- /bin/bash
www-data@nginx-ingress-controller-6bd7c597cb-6pchv:/etc/nginx$ cat nginx.conf
......
    ## start server myapp.magedu.com
    server {
        server_name myapp.magedu.com ;
        
        listen 80;
        
        set $proxy_upstream_name "-";
        
        location / {
            
            set $namespace      "default";
            set $ingress_name   "ingress-myapp";
            set $service_name   "myapp";
            set $service_port   "80";
            set $location_path  "/";
            
            rewrite_by_lua_block {
                
                balancer.rewrite()
                
            }
            
            log_by_lua_block {
                
                balancer.log()
                
                monitor.call()
            }
......
```



## 配置Ingress的TLS支持

实验目标：

- 访问https://tomcat.yfming.com:30443，请求被转发到tomcat服务



1. 准备证书

```
[root@k8s-master ingress]# openssl genrsa -out tls.key 2048 
Generating RSA private key, 2048 bit long modulus
.......+++
.......................+++
e is 65537 (0x10001)

[root@k8s-master ingress]# openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=tomcat.yfming.com
```

2. 生成secret

```
[root@k8s-master ingress]# kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key
secret/tomcat-ingress-secret created
[root@k8s-master ingress]# kubectl get secret
NAME                    TYPE                                  DATA      AGE
default-token-j5pf5     kubernetes.io/service-account-token   3         39d
tomcat-ingress-secret   kubernetes.io/tls                     2         9s
[root@k8s-master ingress]# kubectl describe secret tomcat-ingress-secret
Name:         tomcat-ingress-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1294 bytes
tls.key:  1679 bytes
```

3. 创建tomcat-ingress-tls.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat-ingress-tls
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - tomcat.yfming.com
    secretName: tomcat-ingress-secret
  rules:
  - host: tomcat.yfming.com
    http:
      paths:
      - path:
        backend:
          serviceName: tomcat
          servicePort: 8080
```

4. 测试，访问https://tomcat.yfming.com:30443，由于证书是我们自己签署的，接受不信任证书后，正常响应tomcat首页

## 参考资料
> - []()
> - []()
