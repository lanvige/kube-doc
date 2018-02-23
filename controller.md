    # Controller

kube-controller-manager
Component on the master that runs controllers.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

Node Controller: Responsible for noticing and responding when nodes go down.
Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

按照我们说的 MVVM 模型，会有一个东西不断的监听对应的预定状态并和实际状态进行对比

这个就是 controller。

一共有多少个 controller 呢？

它和 object 是如何对应的呢？

