# 看看 Kubenetes 如何发布

- ReplicaController
- ReplicaSet
- Deployments
- DaemonSet



https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/


## Pod

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```



但通常我们不会直接启动 Pod，因为单一的 Pod 并不是高可用，我们会用多副本的方式来启动


## ReplicaController

Controller 在整个 Kubernetes 中是有着特殊意义的，这里叫这个名字不大好，而后来这个概念也被新的 ReplicaSet 给代替了，所以，这里不再过多介绍。


## ReplicaSet

Replication Controller确保任何时候Kubernetes集群中有指定数量的pod副本(replicas)在运行， 如果少于指定数量的pod副本(replicas)，Replication Controller会启动新的Container，反之会杀死多余的以保证数量不变。Replication Controller使用预先定义的pod模板创建pods，一旦创建成功，pod 模板和创建的pods没有任何关联，可以修改pod 模板而不会对已创建pods有任何影响，也可以直接更新通过Replication Controller创建的pods。对于利用pod 模板创建的pods，Replication Controller根据label selector来关联，通过修改pods的label可以删除对应的pods。

``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```


我们还有另一种方式


## Deployment

这才是我们最常用的方式。

Deployment 为 Pod 和 ReplicaSet 提供声明式更新。

你只需要在 Deployment 中描述您想要的目标状态是什么，Deployment controller 就会帮您将 Pod 和 ReplicaSet 的实际状态改变到您的目标状态。您可以定义一个全新的 Deployment 来创建 ReplicaSet 或者删除已有的 Deployment 并创建一个新的来替换。

Kubernetes Deployment 提供了官方的用于更新 Pod 和 Replica Set（下一代的Replication Controller）的方法，您可以在 Deployment 对象中只描述您所期望的理想状态（预期的运行状态），Deployment 控制器为您将现在的实际状态转换成您期望的状态，例如，您想将所有的 webapp:v1.0.9 升级成 webapp:v1.1.0，您只需创建一个Deployment，Kubernetes 会按照 Deployment 自动进行升级。现在，您可以通过 Deployment 来创建新的资源（pod，rs，rc），替换已经存在的资源等。


下面是Deployment的典型用例：

* 使用Deployment来创建ReplicaSet。ReplicaSet在后台创建pod。检查启动状态，看它是成功还是失败。
* 然后，通过更新Deployment的PodTemplateSpec字段来声明Pod的新状态。这会创建一个新的ReplicaSet，Deployment会按照控制的速率将pod从旧的ReplicaSet移动到新的ReplicaSet中。
* 如果当前状态不稳定，回滚到之前的Deployment revision。每次回滚都会更新Deployment的revision。
* 扩容Deployment以满足更高的负载。
* 暂停Deployment来应用PodTemplateSpec的多个修复，然后恢复上线。
* 根据Deployment 的状态判断上线是否hang住了。
* 清除旧的不必要的 ReplicaSet。

``` yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8 # Update the version of nginx from 1.7.9 to 1.8
        ports:
        - containerPort: 80
```




理论一，层次理论。每一层完成自己层的内容。

Deployment
ReplicaSet

那 ReplicaSet 的意义在哪，也就是如果用 Development 有什么时间不行。非要用底层的它。
Deployment 比它多了什么，有什么用？



* ReplicaSet 全部功能：Deployment继承了上面描述的 ReplicaSet  全部功能。
* 事件和状态查看：可以查看Deployment的升级详细进度和状态。
* 回滚：当升级pod镜像或者相关参数的时候发现问题，可以使用回滚操作回滚到上一个稳定的版本或者指定的版本。
* 版本记录: 每一次对Deployment的操作，都能保存下来，给予后续可能的回滚使用。
* 暂停和启动：对于每一次升级，都能够随时暂停和启动。
* 多种升级方案：Recreate----删除所有已存在的pod,重新创建新的; RollingUpdate----滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数，例如设置最大不可用pod数量，最小升级间隔时间等等。



就是说，如果我们管理一个设备，我们只要知道它自己的信息就行了。

但发布又是一个很宽泛的概念，关于怎么发布，回滚升级，发布状态。这就不再是设备上的属性，或者 Pod 上的属性了。


然后，如果上面的概念解释的通，那余下的就是原理了。

Replication Controller 在其中的工作方式是什么？

