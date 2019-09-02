---
title: 08-Pod控制器-Deployment及ReplicaSet
toc: true
date: 2019-08-31 01:01:51
tags:
categories:
---



Pod控制器是用于实现管理pod的中间层，确保pod资源符合预期的状态，pod的资源出现故障时，会尝试 进行重启，当根据重启策略无效，则会重新新建pod的资源。

## Pod控制器类型

- **ReplicaSet:** 

代用户创建指定数量的pod副本数量，确保pod副本数量符合预期状态，并且支持滚动式自动扩容和缩容功能。ReplicaSet主要三个组件组成：1. 用户期望的pod副本数量，2. 标签选择器，判断哪个pod归自己管理，3. 当现存的pod数量不足，会根据pod资源模板进行新建。

帮助用户管理无状态的pod资源，精确反应用户定义的目标数量，但是RelicaSet不是直接使用的控制器，而是使用Deployment。

- **Deployment：**工作在ReplicaSet之上，用于管理无状态应用，目前来说最好的控制器。支持滚动更新和回滚功能，还提供声明式配置
- **DaemonSet：**用于确保集群中的每一个节点只运行特定的pod副本，通常用于实现系统级后台任务。比如ELK服务，特性：服务是无状态的且服务必须是守护进程	
- **Job：**只要完成就立即退出，不需要重启或重建。
- **Cronjob：**周期性任务控制，不需要持续后台运行
- **StatefulSet：**管理有状态应用

## ReplicaSet控制器

ReplicationController用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的Pod来替代；而如果异常多出来的容器也会自动回收。

在新版本的Kubernetes中建议使用ReplicaSet来取代ReplicationController。ReplicaSet跟ReplicationController没有本质的不同，只是名字不一样，并且ReplicaSet支持集合式的selector。

虽然ReplicaSet可以独立使用，但一般还是建议使用 Deployment 来自动管理ReplicaSet，这样就无需担心跟其他机制的不兼容问题（比如ReplicaSet不支持rolling-update但Deployment支持）。

**ReplicaSet示例：**

```
（1）命令行查看ReplicaSet清单定义规则
[root@k8s-master ~]# kubectl explain rs
[root@k8s-master ~]# kubectl explain rs.spec
[root@k8s-master ~]# kubectl explain rs.spec.template

（2）新建ReplicaSet示例
[root@k8s-master ~]# vim rs-demo.yaml

apiVersion: apps/v1　　#api版本定义
kind: ReplicaSet　　#定义资源类型为ReplicaSet
metadata:　　#元数据定义
    name: myapp
    namespace: default
spec:　　#ReplicaSet的规格定义
    replicas: 2　　#定义副本数量为2个
    selector:　　　　#标签选择器，定义匹配pod的标签
        matchLabels:
            app: myapp
            release: canary
    template:　　#pod的模板定义
        metadata:　　#pod的元数据定义
            name: myapp-pod　　　#自定义pod的名称　
            labels: 　　#定义pod的标签，需要和上面定义的标签一致，也可以多出其他标签
                app: myapp
                release: canary
                environment: qa
        spec:　　#pod的规格定义
            containers:　　#容器定义
            - name: myapp-container　　#容器名称
              image: ikubernetes/myapp:v1　　#容器镜像
              ports:　　#暴露端口
              - name: http
                containerPort: 80

（3）创建ReplicaSet定义的pod                
[root@k8s-master ~]# kubectl create -f rs-demo.yaml
[root@k8s-master ~]# kubectl get pods　　#获取pod信息
[root@k8s-master ~]# kubectl describe pods myapp-***　　#查看pod详细信息

（4）修改pod的副本数量
[root@k8s-master ~]# kubectl edit rs myapp
replicas: 5
[root@k8s-master ~]# kubectl get rs -o wide

（5）修改pod的镜像版本
[root@k8s-master ~]# kubectl edit rs myapp
image: ikubernetes/myapp:v2　　
[root@k8s-master ~]# kubectl delete pods myapp-*** 　　#修改了pod镜像版本，pod需要重建才能达到最新版本
[root@k8s-master ~]# kubectl create -f rs-demo.yaml
```

## Deployment控制器

Deployment为Pod和Replica Set（下一代Replication Controller）提供声明式更新。只需要在 Deployment 中描述想要的目标状态是什么，Deployment controller 就会帮您将 Pod 和ReplicaSet 的实际状态改变到您的目标状态。



## **解析Deployment Spec**

首先看一个官方的nginx-deployment.yaml的例子： 


```
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
        app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

使用kubectl explain deployment.spec查看具体Deployment spec的配置选项，解析如下：

- **Replicas（副本数量）：**

  .spec.replicas 是可以选字段，指定期望的pod数量，默认是1。

- **Selector（选择器）：**
  .spec.selector是可选字段，用来指定 label selector ，圈定Deployment管理的pod范围。如果被指定，.spec.selector 必须匹配 .spec.template.metadata.labels，否则它将被API拒绝。如果  .spec.selector 没有被指定 .spec.selector.matchLabels  默认是.spec.template.metadata.labels。

- **Pod Template（Pod模板）：**
  .spec.template 是 .spec中唯一要求必须的字段。

​	.spec.template 是 pod template. 它跟 Pod有一模一样的schema，除了它是嵌套的并且不需要apiVersion 和 kind字段。

​	另外为了划分Pod的范围，Deployment中的pod template必须指定适当的label（不要跟其他controller重复了，参考selector）和适当的重启策略。

.spec.template.spec.restartPolicy 可以设置为 Always , 如果不指定的话这就是默认配置。

- **strategy（更新策略）：**
  .spec.strategy 指定新的Pod替换旧的Pod的策略。 .spec.strategy.type 可以是"**Recreate**"或者是 "**RollingUpdate**"。**"RollingUpdate"是默认值**。

​	**Recreate：** 重建式更新，就是删一个建一个。类似于ReplicaSet的更新方式，即首先删除现有的Pod对象，然后由控制器基于新模板重新创建新版本资源对象。

​	**rollingUpdate：**滚动更新，简单定义 更新期间pod最多有几个等。可以指定**maxUnavailable** 和 **maxSurge** 来控制 rolling update 进程。

- **maxSurge：**

  `.spec.strategy.rollingUpdate.maxSurge` 是可选配置项，用来指定可以超过期望的Pod数量的最大个数。该值可以是一个绝对值（例如5）或者是期望的Pod数量的百分比（例如10%）。当`MaxUnavailable`为0时该值不可以为0。通过百分比计算的绝对值向上取整。默认值是1。

　　**例如，**该值设置成30%，启动rolling   update后新的ReplicatSet将会立即扩容，新老Pod的总数不能超过期望的Pod数量的130%。旧的Pod被杀掉后，新的ReplicaSet将继续扩容，旧的ReplicaSet会进一步缩容，确保在升级的所有时刻所有的Pod数量和不会超过期望Pod数量的130%。

-  **maxUnavailable：**

  `.spec.strategy.rollingUpdate.maxUnavailable` 是可选配置项，用来指定在升级过程中不可用Pod的最大数量。该值可以是一个绝对值（例如5），也可以是期望Pod数量的百分比（例如10%）。通过计算百分比的绝对值向下取整。  如果`.spec.strategy.rollingUpdate.maxSurge` 为0时，这个值不可以为0。默认值是1。

　　**例如，**该值设置成30%，启动rolling update后旧的ReplicatSet将会立即缩容到期望的Pod数量的70%。新的Pod ready后，随着新的ReplicaSet的扩容，旧的ReplicaSet会进一步缩容确保在升级的所有时刻可以用的Pod数量至少是期望Pod数量的70%。

![](08-Pod控制器-Deployment及ReplicaSet/deploy-rolling.png)

**PS：maxSurge和maxUnavailable的属性值不可同时为0，否则Pod对象的副本数量在符合用户期望的数量后无法做出合理变动以进行更新操作。**

　　**在配置时，用户还可以使用Deployment控制器的spec.minReadySeconds属性来控制应用升级的速度。新旧更替过程中，新创建的Pod对象一旦成功响应就绪探测即被认为是可用状态，然后进行下一轮的替换。而spec.minReadySeconds能够定义在新的Pod对象创建后至少需要等待多长的时间才能会被认为其就绪，在该段时间内，更新操作会被阻塞。**

 

- **revisionHistoryLimit（历史版本记录）：**

  Deployment revision history存储在它控制的ReplicaSets中。默认保存记录10个　　

　　.spec.revisionHistoryLimit  是一个可选配置项，用来指定可以保留的旧的ReplicaSet数量。该理想值取决于心Deployment的频率和稳定性。如果该值没有设置的话，默认所有旧的Replicaset或会被保留，将资源存储在etcd中，是用kubectl  get  rs查看输出。每个Deployment的该配置都保存在ReplicaSet中，然而，一旦删除的旧的RepelicaSet，Deployment就无法再回退到那个revison了。

　　如果将该值设置为0，所有具有0个replica的ReplicaSet都会被删除。在这种情况下，新的Deployment rollout无法撤销，因为revision history都被清理掉了。

**PS：为了保存版本升级的历史，需要再创建Deployment对象时，在命令中使用"--record"选项**

 

- **rollbackTo：**　　　　　　

 　　`.spec.rollbackTo` 是一个可以选配置项，用来配置Deployment回退的配置。设置该参数将触发回退操作，每次回退完成后，该值就会被清除。

-  **revision：**

  `.spec.rollbackTo.revision`是一个可选配置项，用来指定回退到的revision。默认是0，意味着回退到上一个revision。

- **progressDeadlineSeconds：**　　

`　　.spec.progressDeadlineSeconds` 是可选配置项，用来指定在系统报告Deployment的[failed progressing](https://kubernetes.io/docs/concepts/workloads/controllers/deployment.md#failed-deployment)——表现为resource的状态中`type=Progressing`、`Status=False`、 `Reason=ProgressDeadlineExceeded`前可以等待的Deployment进行的秒数。Deployment controller会继续重试该Deployment。未来，在实现了自动回滚后， deployment controller在观察到这种状态时就会自动回滚。

　　如果设置该参数，该值必须大于 `.spec.minReadySeconds`。

- **paused：**

　`.spec.paused`是可以可选配置项，boolean值。用来指定暂停和恢复Deployment。Paused和没有paused的Deployment之间的唯一区别就是，所有对paused  deployment中的PodTemplateSpec的修改都不会触发新的rollout。Deployment被创建之后默认是非paused。　



## **创建Deployment**

```
[root@k8s-master ~]# vim deploy-demo.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-deploy
    namespace: default
spec:
    replicas: 2
    selector:
        matchLabels:
            app: myapp
            release: canary
    template:
        metadata:
            labels: 
                app: myapp
                release: canary
        spec:
            containers:
            - name: myapp
              image: ikubernetes/myapp:v1
              ports:
              - name: http
                containerPort: 80

[root@k8s-master ~]# kubectl apply -f deploy-demo.yaml
[root@k8s-master ~]# kubectl get deploy
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
myapp-deploy       2         0         0            0           1s
[root@k8s-master ~]# kubectl get rs
```

输出结果表明我们希望的repalica数是2（根据deployment中的`.spec.replicas`配置）当前replica数（ `.status.replicas`）是0, 最新的replica数（`.status.updatedReplicas`）是0，可用的replica数（`.status.availableReplicas`）是0。过几秒后再执行`get`命令，将获得如下输出：

```
[root@k8s-master ~]# kubectl get deploy
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
myapp-deploy       2         2         2            2           10s
```

我们可以看到Deployment已经创建了2个 replica，所有的 replica 都已经是最新的了（包含最新的pod template），可用的（根据Deployment中的`.spec.minReadySeconds`声明，处于已就绪状态的pod的最少个数）。执行`kubectl get rs`和`kubectl get pods`会显示Replica Set（RS）和Pod已创建。

```
[root@k8s-master ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
myapp-deploy-2035384211       2         2         0       18s
```

  ReplicaSet 的名字总是`<Deployment的名字>-<pod template的hash值>`。

```
[root@k8s-master ~]# kubectl get pods --show-labels
NAME                            READY     STATUS    RESTARTS   AGE       LABELS
myapp-deploy-2035384211-7ci7o   1/1       Running   0          10s       app=myapp,release=canary,pod-template-hash=2035384211
myapp-deploy-2035384211-kzszj   1/1       Running   0          10s       app=myapp,release=canary,pod-template-hash=2035384211
```

刚创建的Replica Set将保证总是有2个myapp的 pod 存在。

上面示例输出中的 pod label 里的 pod-template-hash label。当 Deployment 创建或者接管 
ReplicaSet 时，Deployment controller 会自动为 Pod 添加 pod-template-hash 
label。这样做的目的是防止 Deployment 的子ReplicaSet 的 pod 名字重复。通过将 ReplicaSet 
的PodTemplate 进行哈希散列，使用生成的哈希值作为 label 的值，并添加到 ReplicaSet selector 里、 pod 
template label 和 ReplicaSet 管理中的 Pod 上。

## 更新升级Deployment

升级有两种方法：分别是通过应用yaml文件升级和直接set命令升级镜像版本升级

**1. 通过直接更改yaml的方式进行升级，如下修改版本号后应用配置：**

```
[root@k8s-master ~]# vim deploy-demo.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-deploy
    namespace: default
spec:
    replicas: 2
    selector:
        matchLabels:
            app: myapp
            release: canary
    template:
        metadata:
            labels: 
                app: myapp
                release: canary
        spec:
            containers:
            - name: myapp
              image: ikubernetes/myapp:v2
              ports:
              - name: http
                containerPort: 80
[root@k8s-master ~]# kubectl apply -f deploy.yaml
```

**升级过程(我们看到，是停止一台，升级一台的这种循环。)**

```
[root@k8s-master ~]# kubectl get pods -l app=myapp -w
NAME                           READY     STATUS    RESTARTS   AGE
myapp-deploy-f4bcc4799-cs5xc   1/1       Running   0          23m
myapp-deploy-f4bcc4799-cwzd9   1/1       Running   0         14m
```

**查看一下 rs的情况，以下可以看到原的rs作为备份，而现在是启动新的rs**

```
[root@k8s-master ~]# kubectl get rs -o wide
NAME                      DESIRED   CURRENT   READY     AGE       CONTAINER(S)       IMAGE(S)               SELECTOR
myapp-deploy-869b888f66   2         2         2         3m        myapp-containers   ikubernetes/myapp:v2   app=myapp,pod-template-hash=4256444922,release=canary
myapp-deploy-f4bcc4799    0         0         0         29m       myapp-containers   ikubernetes/myapp:v1   app=myapp,pod-template-hash=906770355,release=canary
```

**2. 通过set 命令直接修改image的版本进行升级，如下：**

```
[root@k8s-master ~]# kubectl set image deployment/myapp-deploy myapp=ikubernetes/myapp:v2
```

## **扩容Deployment**

扩容有以下3种方式

**1. 使用以下命令扩容 Deployment：**

```
[root@k8s-master ~]# kubectl scale deployment myapp-deploy --replicas 5
```

**2. 直接修改yaml文件的方式进行扩容：**

```
[root@k8s-master ~]# vim demo.yaml
修改.spec.replicas的值
spec:
  replicas: 5
[root@k8s-master ~]# kubectl apply -f demo.yaml
```

**3. 通过打补丁的方式进行扩容：**

```
[root@k8s-master ~]# kubectl patch deployment myapp-deploy -p '{"spec":{"replicas":5}}'
[root@k8s-master ~]# kuebctl get pods
```

## **修改滚动更新策略**

**可以通过打补丁的方式进行修改更新策略，如下：**

```
[root@k8s-master ~]# kubectl patch deployment myapp-deploy -p '{"spec":{"strategy":{"rollingupdate":{"maxsurge“:1,"maxUnavailable":0}}}}'
[root@k8s-master ~]# kubectl describe deploy myapp-deploy
Name:            myapp-deploy
Namespace:        default
CreationTimestamp:    Tue, 28 Aug 2018 09:52:03 -0400
Labels:            app=myapp
            release=canary
Annotations:        deployment.kubernetes.io/revision=4
            kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"myapp-deploy","namespace":"default"},"spec":{"replicas":3,"selector":{...
Selector:        app=myapp,release=dev
Replicas:        3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:        RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:    0 max unavailable, 1 max surge
Pod Template:
....
```

## **金丝雀发布** 

Deployment控制器支持自定义控制更新过程中的滚动节奏，如“暂停(pause)”或“继续(resume)”更新操作。比如等待第一批新的Pod资源创建完成后立即暂停更新过程，此时，仅存在一部分新版本的应用，主体部分还是旧的版本。然后，再筛选一小部分的用户请求路由到新版本的Pod应用，继续观察能否稳定地按期望的方式运行。确定没问题之后再继续完成余下的Pod资源滚动更新，否则立即回滚更新操作。这就是所谓的金丝雀发布（Canary  Release），如下命令演示：

```
（1）更新deployment的v3版本，并配置暂停deployment
[root@k8s-master ~]# kubectl set image deployment myapp-deploy myapp=ikubernetes/myapp:v3 && kubectl rollout pause deployment myapp-deploy    
deployment "myapp-deploy" image updated
deployment "myapp-deploy" paused
[root@k8s-master ~]# kubectl rollout status deployments myapp-deploy　　#观察更新状态

（2）监控更新的过程，可以看到已经新增了一个资源，但是并未按照预期的状态去删除一个旧的资源，就是因为使用了pause暂停命令
[root@k8s-master ~]# kubectl get pods -l app=myapp -w 
NAME                            READY     STATUS    RESTARTS   AGE
myapp-deploy-869b888f66-dpwvk   1/1       Running   0          24m
myapp-deploy-869b888f66-frspv   1/1       Running   0         24m
myapp-deploy-869b888f66-sgsll   1/1       Running   0         24m
myapp-deploy-7cbd5b69b9-5s4sq   0/1       Pending   0         0s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       Pending   0         0s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       ContainerCreating   0         1s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       ContainerCreating   0         2s
myapp-deploy-7cbd5b69b9-5s4sq   1/1       Running   0         19s

（3）确保更新的pod没问题了，继续更新
[root@k8s-master ~]# kubectl rollout resume deploy  myapp-deploy

（4）查看最后的更新情况
[root@k8s-master ~]# kubectl get pods -l app=myapp -w 
NAME                            READY     STATUS    RESTARTS   AGE
myapp-deploy-869b888f66-dpwvk   1/1       Running   0          24m
myapp-deploy-869b888f66-frspv   1/1       Running   0         24m
myapp-deploy-869b888f66-sgsll   1/1       Running   0         24m
myapp-deploy-7cbd5b69b9-5s4sq   0/1       Pending   0         0s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       Pending   0         0s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       ContainerCreating   0         1s
myapp-deploy-7cbd5b69b9-5s4sq   0/1       ContainerCreating   0         2s
```

## **Deployment版本回退**

默认情况下，kubernetes 会在系统中保存前两次的 Deployment 的 rollout 历史记录，以便可以随时回退（您可以修改`revision history limit`来更改保存的revision数）。

注意： 只要 Deployment 的 rollout 被触发就会创建一个 revision。也就是说当且仅当 Deployment 的 Pod template（如`.spec.template`）被更改，例如更新template 中的 label 和容器镜像时，就会创建出一个新的 revision。

其他的更新，比如扩容 Deployment 不会创建 revision——因此我们可以很方便的手动或者自动扩容。这意味着当您回退到历史 revision 时，只有 Deployment 中的 Pod template 部分才会回退。

```
[root@k8s-master ~]#  kubectl rollout history deploy  myapp-deploy　　#检查Deployment升级记录
deployments "myapp-deploy"
REVISION    CHANGE-CAUSE
0        <none>
3        <none>
4        <none>
5        <none>
```



这里在创建deployment时没有增加--record参数，所以并不能看到revision的变化。在创建 Deployment 的时候使用了`--record`参数可以记录命令，就可以方便的查看每次 revision 的变化。

**查看单个revision 的详细信息：**

```
[root@k8s-master ~]# kubectl rollout history deployment/myapp-deploy --revision=2
```

回退历史版本，默认是回退到上一个版本：

```
[root@k8s-master ~]# kubectl rollout undo deployment/myapp-deploy
deployment "myapp-deploy" rolled back
```

也可以使用 `--revision`参数指定某个历史版本：

```
[root@k8s-master ~]# kubectl rollout undo deployment/myapp-deploy --to-revision=2
deployment "myapp-deploy" rolled back
```

## 参考资料

> - []()
> - []()
