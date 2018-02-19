# DaemonSet

因为很多时候，我们需要在机器上启动一信唯一的进程，用来监听一些事件。

像 启动 Proxy，启动日志监听。


## 什么是DaemonSet

DaemonSet能够让所有（或者一些特定）的Node节点运行同一个pod。当节点加入到kubernetes集群中，pod会被（DaemonSet）调度到该节点上运行，当节点从kubernetes集群中被移除，被（DaemonSet）调度的pod会被移除，如果删除DaemonSet，所有跟这个DaemonSet相关的pods都会被删除。

在使用kubernetes来运行应用时，很多时候我们需要在一个区域（zone）或者所有Node上运行同一个守护进程（pod），例如如下场景：

每个Node上运行一个分布式存储的守护进程，例如glusterd，ceph
运行日志采集器在每个Node上，例如fluentd，logstash
运行监控的采集端在每个Node，例如prometheus node exporter，collectd等
在简单的情况下，一个DaemonSet可以覆盖所有的Node，来实现Only-One-Pod-Per-Node这种情形；在有的情况下，我们对不同的计算几点进行着色，或者把kubernetes的集群节点分为多个zone，DaemonSet也可以在每个zone上实现Only-One-Pod-Per-Node。


https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.



NodeIP and Known Port: Pods in the DaemonSet can use a hostPort, so that the pods are reachable via the node IPs. Clients know the list of node IPs somehow, and know the port by convention.


## 参数说明


## Port 绑定

DaemonSet 最常被用到的就是端口绑定。我们也基于这个功能，将 Proxy Servier 用这种方式来部署。

- hostNetwork: true
- containerPort: 80
- hostPort: 80


hostNetwork=true, 将pod中所有容器的端口号直接映射到物理机上

* 如果不指定 hostPort，默认 hostPort 等于 containerPort，但如果要单独指定，他们两个必须相同？？？？

> DaemonSet.apps "traefik-ingress-controller" is invalid: spec.template.spec.containers[0].ports[1].containerPort: Invalid value: 8080: must match `hostPort` when `hostNetwork` is true


### 这是一个 traefik 的 demo：

``` yaml
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      containers:
      - image: traefik:v1.5.0-alpine
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8088
        securityContext:
          privileged: true
        args:
        - -d
        - --api
        - --kubernetes
```

