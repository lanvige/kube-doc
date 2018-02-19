# Kubernetes 入门文章集

![](assets/kube-logo.png)



这是个人学习 Kubernets 的一系列总结文章。


## 从 Docker 谈起

从 15 年开始， 容器就开始成为最火的名词，Docker 就是容器的代名词。

这也激发了应用架构的一次变革，从单体架构，向微服务架构转移。

而到了 2017 年，最惨科技公司排行最靠前的竟然是这家公司，这也说明了一件事，容器已经到达了顶峰。


## 为什么是 Kubernets

谈起 Kubernets，还是要再回看头下 Docker，它作为容器，只解决了应用运行时的问题，但应用的编排和网络的问题，都没有很好的方案。

这就是 Kubernetes 所要做的。


[详情](docker.md)

[详情](etcd.md)

### Master / Worker Node

[详情](node.md)

### kube-proxy

### kubelet

### 

<br />

## 概念

Kubernets 解决了很多的问题，但也提出了很多的概念。它可比 Docker 要复杂的多。

### Pod

Kubernetes 在容器的概念上又进行了一次封装，叫 Pod，它认为单的的容器不能很好的完成一些指定的工作。而 Pod 则可以把多个容器放在一起进行调度，在同一个 Pod 间的容器可以更好的通讯。

[详情](pod.md)

### Service

Pod 是动态的，它是 Kubernetes 扩容缩容的调度单位，不能直接对外提供

Service 对外提供服务有三种形式

- NodePort
- Ingress
- LoadBlance

关于 Service 和对外服务的几种不同之处，请见：[详情](service.md)


### Ingress

Ingress 相对于 NodePort 比较复杂，我们单独拿出来讲一下：

[详情](ingress.md)


### deployment

如何发布一个 启动一个 Pod 呢？有很多种方法，可以单独的发布 Pod，也可以由 ReplicaSet 进行。也可以用 Deployment 的方式。

这是最推荐的方式。



### 对象

对象是 Kubernetes 中的一个概念，它把所有的东西都定义为对象，对象有两种状态，一种是定义的，一种是实际的。

Controller Manager 就是把这两种状态进行校正的的后台服务。

[详情](object.md)


### Controller Manager

上面介绍了这些概念，它们是怎么运转的呢？

它们是由 Controller Manager 来进行执行的。

常见的 Controller Manager 有：

- Node Controller
- Replication Controller
- Endpoints Controller
- Service Account & Token Controllers

[详情](controller.md)



<br>

## 网络 / Network

网络是 Kubernetes 的基石，没有网络，就没有 Kubernetes 的一切。

### CNI

[详情](cni.md)

### Flannel

[详情](flannel.md)

### Calico

[详情](calico.md)



<br>

## 安全


### Account 

[详情](account.md)

### RBAC

[详情](rbac.md)


### Service Account

[详情](service-account.md)

## Other


### Dashboard

Kubernetes 提供了一个好的方式来查看当前的状态。就是 kube-dashboard

[详情](dashboard.md)


