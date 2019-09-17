---
title: 03-集群监控方案：InfluxDB-Heapster-Grafana
toc: true
date: 2019-08-29 11:00:41
tags:
categories:
---

heapster是一个监控计算、存储、网络等集群资源的工具，以k8s内置的cAdvisor作为数据源收集集群信息，并汇总出有价值的性能数据(Metrics)：cpu、内存、网络流量等，然后将这些数据输出到外部存储，如InfluxDB，最后就可以通过相应的UI界面显示出来，如grafana。 另外heapster的数据源和外部存储都是可插拔的，所以可以很灵活的组建出很多监控方案，如：Heapster+ElasticSearch+Kibana等等。



## 部署InfluxDB

- 下面我们使用NotePort暴露monitoring-influxdb服务在主机的31001端口上

- InfluxDB服务端的地址：http://[host-ip]:31001 
- 该地址是heapster写入监控数据的地址
- 该地址是grafana展示监控数据时使用的数据源

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: influxdb
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: mathlsj/heapster-influxdb-amd64:v1.3.3
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
      volumes:
      - name: influxdb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - nodePort: 31001
    port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
```



## 部署Heapster

- 注意修改下面influxDB的地址

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: heapster
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: wjun/heapster-amd64:v1.4.2
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes
        - --sink=influxdb:http://192.168.11.129:31001  # 这里填写刚刚记录下的InfluxDB服务端的地址。
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  selector:
    k8s-app: heapster
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: heapster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: heapster
  namespace: kube-system
```

- --source 为heapster指定获取集群信息的数据源。如果heapster部署在要监控的集群中，如上配置，其他请参考：<https://github.com/kubernetes/heapster/blob/master/docs/source-configuration.md>
- --sink 为heaster指定后端存储，这里我们使用InfluxDB，其他请参考：<https://github.com/kubernetes/heapster/blob/master/docs/sink-owners.md>



## 部署Grafana

- 下面我们使用NotePort暴露monitoring-grafana服务在主机的30108上
- 部署完后修改数据源为上面influxDB的地址，之后就可以查看监控数据

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      task: monitoring
      k8s-app: grafana
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: recotone/heapster-grafana-amd64:v4.4.3
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certificates
          readOnly: true
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GF_SERVER_HTTP_PORT
          value: "3000"
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          value: /
      volumes:
      - name: ca-certificates
        hostPath:
          path: /etc/ssl/certs
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-grafana
  name: monitoring-grafana
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - nodePort: 30108
    port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
```



## QA

1. Grafana读不到监控数据，且kubernetes-dashboard没有cpu和内存监控图像

每个节点加如下参数后重启
```
$ vim /etc/sysconfig/kubelet 

KUBELET_EXTRA_ARGS="--fail-swap-on=false --read-only-port=10255"

$ systemctl restart kubelet
```



## 参考资料
> - []()
> - []()
