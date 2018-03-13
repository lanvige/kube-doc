# namespace

在一个 Kubernetes 集群中，会有不止一个用户，也会有很多的项目，如何进行资源上的分配和隔离呢？

这就是 namespace 要做的，它用来把集群进行隔离。

Namespace中创建的resources/objects都是唯一的，不会跨命名空间。


### 默认 namespace

系统提供两个默认Namespace：kube-system 和 default。

- kube-system 用来放置 Kubernetes 系统的组件
- default 会用来放一些属于其它 Namespace 的对象，项目默认情况下是会连接到 default 命名空间。
- kube-public 是一个特殊的 namespace，可以被所有的用户读，一般用于特殊情况比如初始化一个集群。



### 限制

通过使用资源配额（Resource Quotas）来限制每个Namespace的资源。



