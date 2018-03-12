# kube-dns



## 服务发现

kubernetes 提供了 service 的概念可以通过 VIP 访问 pod 提供的服务，但是在使用的时候还有一个问题：怎么知道某个应用的 VIP？比如我们有两个应用，一个 app，一个 是 db，每个应用使用 rc 进行管理，并通过 service 暴露出端口提供服务。app 需要连接到 db 应用，我们只知道 db 应用的名称，但是并不知道它的 VIP 地址。

最简单的办法是从 Kubernetes 提供的 API 查询。但这是一个糟糕的做法，首先每个应用都要在启动的时候编写查询依赖服务的逻辑，这本身就是重复和增加应用的复杂度；其次这也导致应用需要依赖 kubernetes，不能够单独部署和运行（当然如果通过增加配置选项也是可以做到的，但这又是增加负责度）。

开始的时候，kubernetes 采用了 docker 使用过的方法——环境变量。每个 pod 启动时候，会把通过环境变量设置所有服务的 IP 和 port 信息，这样 pod 中的应用可以通过读取环境变量来获取依赖服务的地址信息。这种方式服务和环境变量的匹配关系有一定的规范，使用起来也相对简单，但是有个很大的问题：依赖的服务必须在 pod 启动之前就存在，不然是不会出现在环境变量中的。

更理想的方案是：应用能够直接使用服务的名字，不需要关心它实际的 ip 地址，中间的转换能够自动完成。名字和 ip 之间的转换就是 DNS 系统的功能，因此 kubernetes 也提供了 DNS 方法来解决这个问题。



## DNS 服务。

服务DNS 服务不是独立的系统服务，而是作为插件来安装的，叫做 addon，不是 kubernetes 集群必须的。可以把它看做运行在集群上的应用，只不过这个应用比较特殊而已。



它应该是一个 controller，用来监控一些数值，来影响自身的配置。

确实，但它不像其它的 controller ，是通过先前的配置安装在 host 上，以 daemon 来启动的。


- https://github.com/kubernetes/dns

Service 默认的注册名：

```
<service_name>.<namespace>.svc.<domain>  
```


## 它经历过三个时期：


DNS 有三种配置方式，在 1.3 之前使用 kube2sky + skydns + etcd  的方式，在 [1.3](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.3.md) 之后可以使用 kubedns + dnsmasq 的方式，而在 [1.9](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md#coredns) 时，CoreDNS (CNCF) 已经冒出。


1.3 时代，带来了 [add-on](https://github.com/kubernetes/kubernetes/pull/25986)


* kube2sky + skydns + etcd
* kube-dns + dnsmasq
* CoreDNS


### kube2sky 模式

这种模式已经不常用了。所以也只是简单的介绍下：

* https://github.com/gravitational/kube2sky
* https://github.com/skynetservices/skydns

这种模式下主要有三个容器在运行：

```
[root@localhost ~]# docker ps

CONTAINER ID        IMAGE              COMMAND                  CREATED             STATUS              PORTS     NAMES
919cbc006da2        gc/kube2sky:1.12   "/kube2sky /kube2sky "   About an hour ago   Up About an hour              k8s_kube2sky
73dd11cac057        etcd:live          "etcd -data-dir=/var/"   About an hour ago   Up About an hour              k8s_etcd
0b10ae639989        skydns:20150703    "bootstrap.sh"           About an hour ago   Up About an hour              k8s_skydns
```


这三个容器的作用分别是：

  
* etcd：保存所有的 DNS 数据
* kube2sky： 通过 kubernetes API 监听 Service 的变化，然后同步到 etcd
* skyDNS：根据 etcd 中的数据，对外提供 DNS 查询服务


### kubeDNS 模式


这是我们当下 1.8，标配的模式，虽然 CoreDNS 声势很大，但目前还不是主要推荐。

* https://github.com/kubernetes/dns
* https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns

这种模式下，kubeDNS 容器替代了原来的三个容器的功能，它会监听 apiserver 并把所有 service 和 endpoints 的结果在内存中用合适的数据结构保存起来，并对外提供 DNS 查询服务。



  
* kubeDNS：提供了原来 kube2sky + etcd + skyDNS 的功能，可以单独对外提供 DNS 查询服务
  
* dnsmasq： 一个轻量级的 DNS 服务软件，可以提供 DNS 缓存功能。kubeDNS 模式下，dnsmasq 在内存中预留一块大小（默认是 1G）的地方，保存当前最常用的 DNS 查询记录，如果缓存中没有要查找的记录，它会到 kubeDNS 中查询，并把结果缓存起来
每种模式都可以运行额外的 exec-healthz 容器对外提供 health check 功能，证明当前 DNS 服务是正常的。
  
* exec-healthz：运行某个命令，根据结果来对外提供 /healthz 结果


#### 实现

kube-dns 将分别为 service 和 pod 生成不同格式的 DNS 记录。

Service

* A记录：生成 `my-svc.my-namespace.svc.cluster.local` 域名，解析成 IP 地址，分为两种情况：
    * 普通 Service：解析成 ClusterIP
    * Headless Service：解析为指定 Pod 的 IP 列表
* SRV 记录：为命名的端口（普通 Service 或 Headless Service）生成 `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local` 的域名

Pod

* A记录：生成域名 pod-ip.my-namespace.pod.cluster.local



#### 安装

参考： https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/kube-dns.yaml.in

它用了这两个镜像：

```
image: gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.8
image: gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.8
```

> dsnmasq 可不是必须的。它是为了提升效率而存在的。它的作用是通过 cache 来加入。


### CoreDNS

![](assets/coredns.png)

* https://coredns.io/
* https://github.com/coredns


> CoreDNS: DNS and Service Discovery

CoreDNS，它采用更模块化，可扩展的框架构建。 Infoblox已经与Miek合作，将此DNS服务器作为Kube-DNS的替代品。


在 Kubernetes 1.9 版本中，已经开始将 CoreDNS 作为备选项了，而且可以预见，在未来，这才是主流的方式。

CoreDNS作为CNCF中的托管的一个项目，在Kuberentes1.9版本中，使用kubeadm方式安装的集群可以通过以下命令直接安装CoreDNS。

```
kubeadm init --feature-gates=CoreDNS=true
```


您也可以使用CoreDNS替换Kubernetes插件kube-dns，可以使用 Pod 部署也可以独立部署，请参考Using CoreDNS for Service Discovery，下文将介绍如何配置kube-dns。


- https://kubernetes.io/docs/tasks/administer-cluster/coredns/
- https://coredns.io/2017/05/08/custom-dns-entries-for-kubernetes/



#### 其它

```
image: coredns/coredns:1.0.4
```


你看，它就一个容器就搞定了。


