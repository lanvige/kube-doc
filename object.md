
# 认识 Kubernetes 中的对象

Kubernetes 是一个完善的系统，这包括了它对各个要件进行了完美的抽象，给这些要件起上漂亮的名字，做好分类，再配上操作它们的方式。

这些要件就是要谈及的对象。


<br>

## 对象分类

我们对对象进行一个分类，让其更加明晰：

| 类别 | 名称 |
| :-: | --- |
| 资源对象 | Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling |
| 配置对象 | Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、ThirdPartyResource、 ServiceAccount |
| 存储对象 | Volume、Persistent Volume |
| 策略对象 | SecurityContext、ResourceQuota、LimitRange |


## 理解 kubernetes 中的对象

在 Kubernetes 系统中，Kubernetes 对象是持久化的条目。Kubernetes 使用这些条目去表示整个集群的状态。特别地，它们描述了如下信息：

* 什么容器化应用在运行（以及在哪个 Node 上）
* 可以被应用使用的资源
* 关于应用如何表现的策略，比如重启策略、升级策略，以及容错策略


### 对象 Spec 与状态

每个 Kubernetes 对象包含两个对象字段，它们负责管理对象的配置：

- 对象 SPEC  （就是 YAML 中的定义）
- 对象 Status （Status 就是实际运行的状态）


SPEC 由用户提供，它描述了对象的 `期望状态` - 希望对象所具有的特征。
Status 描述了对象的 `实际状态`，它是由 Kubernetes 系统提供和更新。

Kubernetes Controller Management 来实现或管理着对象的实际状态以与我们所期望的状态相匹配。


> 单向事件流，用 Flux 架构中的这个说法更能形象的描述这种机制，用户负责定义 Spec，这就是期待的结果，而 Controller 则负责按照 Sepc 去动态执行，将目标结果 Status 变成 Spec 所定义的。



### SPEC 示例：

一个 SPEC 的 Demo，关于 [SPEC 详见](spec.md)：

``` yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
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

### Kubernetes 对象是 “目标性记录”

一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。通过创建对象，可以有效地告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的，这就是 Kubernetes 集群的 期望状态。

操作 Kubernetes 对象（包括创建、修改，或者删除），需要通过 Kubernetes API 来进行，kubectl 也是进行的 调用，也可以通过 SDK 来进行客户端的开发。


