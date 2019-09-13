---
title: kubernetes的网络模型
toc: true
date: 2019-08-30 12:06:42
tags:
categories:
---



Kubernetes设计了一种网络模型，要求无论容器运行在集群中的哪个节点，所有容器都能通过一个扁平的网络平面进行通信，即在同一IP网络中。需要注意的是：在K8S集群中，IP地址分配是以Pod对象为单位，而非容器，同一Pod内的所有容器共享同一网络名称空间，即通过localhost访问。

## Docker网络模型

Docker容器的原生网络模型主要有3种：Bridge（桥接）、Host（主机）、none。

- Bridge：借助虚拟网桥设备为容器建立网络连接。
- Host：设置容器直接共享当前节点主机的网络名称空间。
- none：多个容器共享同一个网络名称空间，仅有lo网卡。

```
#使用以下命令查看docker原生的三种网络
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0efec019c899        bridge              bridge              local
40add8bb5f07        host                host                local
ad94f0b1cca6        none                null                local

#none网络，在该网络下的容器仅有lo网卡，属于封闭式网络，通常用于对安全性要求较高并且不需要联网的应用
[root@localhost ~]# docker run -it --network=none busybox
/ # ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

#host网络，共享宿主机的网络名称空间，容器网络配置和host一致，但是存在端口冲突的问题
[root@localhost ~]# docker run -it --network=host busybox
/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:0c:29:69:a7:23 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.4/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 84129sec preferred_lft 84129sec
    inet6 fe80::20c:29ff:fe69:a723/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue 
    link/ether 02:42:29:09:8f:dd brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:29ff:fe09:8fdd/64 scope link 
       valid_lft forever preferred_lft forever
/ # hostname
localhost

#bridge网络，Docker安装完成时会创建一个名为docker0的linux bridge，不指定网络时，创建的网络默认为桥接网络，都会桥接到docker0上。
[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
docker0     8000.024229098fdd   no      

[root@localhost ~]# docker run -d nginx #运行一个nginx容器
c760a1b6c9891c02c992972d10a99639d4816c4160d633f1c5076292855bbf2b

[root@localhost ~]# brctl show      
bridge name bridge id       STP enabled interfaces
docker0     8000.024229098fdd   no      veth3f1b114

一个新的网络接口veth3f1b114桥接到了docker0上，veth3f1b114就是新创建的容器的虚拟网卡。进入容器查看其网络配置：
[root@localhost ~]# docker exec -it c760a1b6c98 bash
root@c760a1b6c989:/# apt-get update
root@c760a1b6c989:/# apt-get iproute
root@c760a1b6c989:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

 从上可以看到容器内有一个网卡`eth0@if39`，实际上`eth0@if39`和`veth3f1b114`是一对`veth pair`。`veth pair`是一种成对出现的特殊网络设备，可以想象它们由一根虚拟的网线进行连接的一对网卡，`eth0@if39`在容器中，`veth3f1b114`挂在网桥`docker0`上，最终的效果就是`eth0@if39`也挂在了`docker0上`。

  桥接式网络是目前较为流行和默认的解决方案。但是这种方案的弊端是无法跨主机通信的，仅能在宿主机本地进行，而解决该问题的方法就是NAT。所有接入到该桥接设备上的容器都会被NAT隐藏，它们发往Docker主机外部的所有流量都会经过源地址转换后发出，并且默认是无法直接接受节点之外的其他主机发来的请求。当需要接入Docker主机外部流量，就需要进行目标地址转换甚至端口转换将其暴露在外部网络当中。如下图：

![]()

 容器内的属于私有地址，需要在左侧的主机上的eth0上进行源地址转换，而右侧的地址需要被访问，就需要将eth0的地址进行NAT转换。SNAT---->DNAT

 这样的通信方式会比较麻烦，从而需要借助第三方的网络插件实现这样的跨主机通信的网络策略。

## Kubernetes的网络通信类型

在K8S上的网络通信包含以下几类：

- 容器间的通信：同一个Pod内的多个容器间的通信，它们之间通过lo网卡进行通信。
- Pod之间的通信：通过Pod IP地址进行通信。
- Pod和Service之间的通信：Pod IP地址和Service IP进行通信，两者并不属于同一网络，实现方式是通过IPVS或iptables规则转发。
- Service和集群外部客户端的通信，实现方式：Ingress、NodePort、Loadbalance



在k8s集群各节点上，docker0上默认使用同一个子网，不同节点的容器都有可能会获取到相同的地址，那么在跨节点通信时就会出现地址冲突的问题。如果配置为使用不同的子网，也会因为没有准确的路由信息导致无法准确送达报文。所以k8s重新定义了网络模型。

K8S网络的实现不是集群内部自己实现，而是依赖于第三方网络插件----CNI（Container Network Interface）。flannel、calico、canel等是目前比较流行的第三方网络插件实现。这三种的网络插件需要实现Pod网络方案的方式通常有以下几种：虚拟网桥、多路复用（MacVLAN）、硬件交换（SR-IOV）。

无论是上面的哪种方式在容器当中实现，都需要大量的操作步骤，而K8S支持CNI插件进行编排网络，以实现Pod和集群网络管理功能的自动化。每次Pod被初始化或删除，kubelet都会调用默认的CNI插件去创建一个虚拟设备接口附加到相关的底层网络，为Pod去配置IP地址、路由信息并映射到Pod对象的网络名称空间。

在配置Pod网络时，kubelet会在默认的/etc/cni/net.d/目录中去查找CNI  JSON配置文件，然后通过type属性到/opt/cni/bin中查找相关的插件二进制文件，如下面的"portmap"。然后CNI插件调用IPAM插件（IP地址管理插件）来配置每个接口的IP地址：

```
[root@k8s-master ~]# cat /etc/cni/net.d/10-flannel.conflist 
{
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

CNI主要是定义容器网络模型规范，链接容器管理系统（如k8s）和网络插件实现（如flannel），两者主要通过上面的JSON格式文件进行通信，实现容器的网络功能。

CNI的主要核心是：在创建容器时，先创建好网络名称空间（netns），然后调用CNI插件为这个netns配置网络，最后在启动容器内的进程。

 常见的CNI网络插件包含以下几种：

- Flannel：为Kubernetes提供叠加网络的网络插件，基于TUN/TAP隧道技术，使用UDP封装IP报文进行创建叠 加网络，借助etcd维护网络的分配情况，缺点：无法支持网络策略访问控制。
- Calico：基于BGP的三层网络插件，也支持网络策略进而实现网络的访问控制；它在每台主机上都运行一个虚拟路由，利用Linux内核转发网络数据包，并借助iptables实现防火墙功能。实际上Calico最后的实现就是将每台主机都变成了一台路由器，将各个网络进行连接起来，实现跨主机通信的功能。
- Canal：由Flannel和Calico联合发布的一个统一网络插件，提供CNI网络插件，并支持网络策略实现。
- 其他的还包括Weave Net、Contiv、OpenContrail、Romana、NSX-T、kube-router等等。而Flannel和Calico是目前最流行的选择方案。

## Flannel网络插件

在创建集群时，需要指定pod的网段，一般为： 10.244.0.0/16

针对各节点的pod网络，Flannel自动为每个节点分配一个子网，如10.244.1.0/24和10.244.2.0/24，并将分配信息保存在etcd持久存储。然后需要解决两个问题，单节点和跨节点的pod间通信问题。

单节点pod间通信类似于Docker引擎的桥接模式，查看各个节点上的网络接口可以发现多了一个虚拟接口cni0，它是由flanneld创建的一个虚拟网桥，在Pod本地通信使用，仅作用于本地通信。flanneld为每个Pod创建一对veth虚拟设备，一端放在容器接口上，一端放在cni0桥上。 类似于Docker引擎的docker0网桥。

跨节点pod通信，Flannel是采用不同类型的后端网络模型进行处理。其后端的类型有以下几种：

- VxLAN：使用内核中的VxLAN模块进行封装报文。默认就是VXLAN模式，即Overlay Network。
- host-gw：即Host GateWay，通过在节点上创建目标容器地址的路由直接完成报文转发，要求各节点必须在同一个2层网络，对报文转发性能要求较高的场景使用。
- UDP：使用普通的UDP报文封装完成隧道转发。

## VxLAN

flannel运行后，在各Node宿主机多了一个网络接口：flannel.1。它是专门用来封装隧道协议的。

那么跨主机pod通信是如何实现的呢？ master上查看路由表信息：

```
[root@k8s-master ~]# ip route
......
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink 
......
```

发送到`10.244.1.0/24`和`10.244.2.0/24`网段的数据报文发给本机的flannel.1接口，即进入二层隧道，然后对数据报文进行封装（封装VxLAN首部-->UDP首部-->IP首部-->以太网首部），到达目标Node节点后，由目标Node上的flannel.1进行解封装。

VXLAN是Linux内核本身支持的一种网络虚拟化技术，是内核的一个模块，在内核态实现封装解封装，构建出覆盖网络，其实就是一个由各宿主机上的Flannel.1设备组成的虚拟二层网络。

## host-gw

由于VXLAN由于额外的封包解包，导致其性能较差，所以Flannel就有了host-gw模式，即把宿主机当作网关，除了本地路由之外没有额外开销，性能和calico差不多，由于没有叠加来实现报文转发，这样会导致路由表庞大。因为一个节点对应一个网络，也就对应一条路由条目。

host-gw虽然VXLAN网络性能要强很多，但是种方式有个缺陷：要求各物理节点必须在同一个二层网络中，即物理节点必须在同一网段中。这样会使得一个网段中的主机量会非常多，万一发一个广播报文就会产生干扰。在私有云场景下，宿主机不在同一网段是很常见的状态，所以就不能使用host-gw了。

## direct routing

 VXLAN还有另外一种功能，VXLAN也支持类似host-gw的玩法，如果两个节点在同一网段时使用host-gw通信，如果不在同一网段中，即当前pod所在节点与目标pod所在节点中间有路由器，就使用VXLAN这种方式，使用叠加网络。 

结合了Host-gw和VXLAN，这就是VXLAN的**Direct routing模式**。

- Direct routing模式配置

```
修改kube-flannel.yml文件，将flannel的configmap对象改为：

[root@k8s-master ~]# vim kube-flannel.yml 
......
 net-conf.json: |
    {
      "Network": "10.244.0.0/16",   #默认网段
      "Backend": {
        "Type": "vxlan",
        "Directrouting": true   #增加
      }
    }
......

[root@k8s-master ~]# kubectl apply -f kube-flannel.yml 
clusterrole.rbac.authorization.k8s.io/flannel configured
clusterrolebinding.rbac.authorization.k8s.io/flannel configured
serviceaccount/flannel unchanged
configmap/kube-flannel-cfg configured
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created

#查看路由信息
[root@k8s-master ~]# ip route
......
10.244.1.0/24 via 192.168.56.12 dev eth0 
10.244.2.0/24 via 192.168.56.13 dev eth0 
......
```

从上面的结果可以看到，发往`10.244.1.0/24`和`10.244.2.0/24`的包都是直接经过`eth0`网络接口直接发出去的，这就是Directrouting。如果两个节点是跨网段的，则flannel自动降级为VxLAN模式。

此时，在各个集群节点上执行“`iptables-nL”`命令可以看到，iptablesfilter表的FORWARD链上由其生成了如下两条转发规则，它显式放行了`10.244.0.0/16`网络进出的所有报文，用于确保由物理接口接收或发送的目标地址或源地址为`10.244.0.0/16`网络的所有报文均能够正常通行。这些是DirectRouting模式得以实现的必要条件：

```
target      prot    opt     source              destination 
ACCEPT      all     --  10. 244. 0. 0/ 16       0. 0. 0. 0/ 0 
ACCEPT      all     --  0. 0. 0. 0/ 0           10. 244. 0. 0/ 16
```

## 参考资料
> - []()
> - []()
