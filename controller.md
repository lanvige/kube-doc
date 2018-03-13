# Controller Manager


## kube-controller-manager

在 [Object](object.md) 中，我们提到了单向事件流，Controller Manager 会负责根据 Spec 来创建对应的 Status。

这就是 `kube-controller-manager`，Component on the master that runs controllers.

理论上，每个 Controller 应该会用独立的进程，但是这样就很难管理，于是把它们都集中到了一起放在一个启动文件中，并且在同一个进程中来处理了。


## 常见的有以下四种 Controller。

- Node Controller: Responsible for noticing and responding when nodes go down.
- Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
- Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.


> 还需要进行一下更深层次的分析，确认下更深层次的工作原理。



