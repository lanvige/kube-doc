# node

https://kubernetes.io/docs/concepts/architecture/nodes/

A node is a worker machine in Kubernetes, previously known as a minion. A node may be a VM or physical machine, depending on the cluster. Each node has the services necessary to run pods and is managed by the master components. The services on a node include Docker, kubelet and kube-proxy. See The Kubernetes Node section in the architecture design doc for more details.



## Kubernets 属于主从的分布式集群架构，包含 Master 和 Node：

Master 作为控制节点，调度管理整个系统，包含以下组件：

* API Server 作为 kubernetes 系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd(一个分布式强一致性的key/value存储)。
* Scheduler：负责集群的资源调度，为新建的pod分配机器。这部分工作分出来变成一个组件，意味着可以很方便地替换成其他的调度器。
* Controller Manager：负责执行各种控制器，目前有两类：
    * Endpoint Controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
    * Replication Controller：定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。

Node 是运行节点，运行业务容器，包含以下组件：

* Kubelet：
责管控docker容器，如启动/停止、监控运行状态等。它会定期从etcd获取分配到本机的pod，并根据pod信息启动或停止相应的容器。同时，它也会接收 apiserver 的 HTTP 请求，汇报 pod 的运行状态。

* Kube-Proxy：
负责为pod提供代理。它会定期从etcd获取所有的service，并根据service信息创建代理。当某个客户pod要访问其他pod时，访问请求会经过本机proxy做转发。

Kubernets使用Etcd作为存储和通信中间件，实现Master和Node的一致性，这是目前比较常见的做法，典型的SOA架构，解耦Master和Node。



## Node Status

A node’s status contains the following information:

* Addresses
* Phase deprecated
* Condition
* Capacity
* Info



节点并非Kubernetes创建，而是由云平台创建，或者就是物理机器、虚拟机。在Kubernetes中，节点仅仅是一条记录，节点创建之后，Kubernetes会检查其是否可用。在Kubernetes中，节点用如下结构保存：

``` json
{
  "id": "10.1.2.3",
  "kind": "Minion",
  "apiVersion": "v1beta1",
  "resources": {
    "capacity": {
      "cpu": 1000,
      "memory": 1073741824
    },
  },
  "labels": {
    "name": "my-first-k8s-node",
  },
}
```