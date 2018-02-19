# Service 的概念

Service 是整个体系中最重要的概念之一，但它也相对于比较难于理解，因为它只是一个概念，而非一个实体。

它是由 kube-proxy 来实现的。




``` yaml
```

# Kubernetes Service 对外发布 Service

Service 的虚拟 IP 是有 Kubernetes 虚拟出来的内部网络，外部网络是无法访问的，但是有一些 Service 有需要对外暴露，比如 Web 前段。这时候就需要增加一层路由转发，即外网到内网的转发，Kubernetes 提供了 NodePort Service 、LoadBalancer Service 和 Lngress 可以发布 Service。

## NodePort Service

NodePort Service 是类型为 NodePort 的 Service，Kubernetes 出去会分配给 NodePort Service 一个内部虚拟IP，另外会在每一个 Node 上暴露端口 NodePort，外部网络可以通过 [NodeIP]:[NodePort] 访问到 Service。创建一个 NodePort Service nodeport-nginx-service.yaml：

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
    #nodePort: 8081
    protocol: TCP
```

nodePort：可以固定端口，默认会从30000~32000 端口随机开设。
创建：

```
$ kubectl create -f nodeport-nginx-service.yaml
```

创建成功后查询：

``` yaml my-nginx-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  #hostNetwork: true # 可以设置 Pod 默认级别，这支此选项后，hostPort默认等于 containerPort，Pod 上所有的 端口都被映射到 Node 上。
  replicas: 3
  selector:
    app: my-nginx
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          #hostPort: 81 可以自定义端口到 node 上
```
 
- hostNetwork： 可以设置 Pod 默认级别，这支此选项后，hostPort默认等于 containerPort，Pod 上所有的 端口都被映射到 Node 上。
- hostPort：可以自定义端口到 node 上。

```
$ kubectl describe service my-nginx 
```

![](https://wiki.shileizcc.com/download/attachments/2523169/image2016-8-4%2016%3A53%3A12.png?version=1&modificationDate=1470300793000&api=v2)


可以看到，Kubernetes 给 NodePort Service 中每一个端口都创建了一个 NodePort （http 30433/TCP）,在 NodePort Service 定义中可以通过 ,spec.ports[].nodePort 指定固定 NodPort，NodePort 的范围默认是 30000 ~ 32767，可以通过 Kubernetes API Service 的启动参数 --service-node-port-range 指定范围。NodePort Service 就可以通过[NodeIP]:[NodePort] 访问,而当 NodeIP 是一个公网 IP 时，外部就可以访问到 NodePort Service 了。
 
## LoadBalancer Service

LoadBalancer Service 是类型为 LoadBalancer 的 Service，LoadBalancer Service 是建立在 NodePort Service 集群上的，Kubernetes 会分配给 LoadBalancer Service 一个内部的虚拟 IP，并且暴露 NodePort。除此之外，Kubernetes 请求底层云平台创建一个负载均衡器，将每个 Node 作为后端，负载均衡器将会转发请求到 [NodeIP]:[NodePort]。

LoadBalancer Service 需要底层云平台支持创建负载均衡其，比如 GCE，现在创建一个 LoadBalancer Service， 

``` yaml loadbalancer-nginx-service.yaml :
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
  selector:
    app: my-nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  type: LoadBalancer
```
  
```
$ kubectl create -f loadbalancer-nginx-service.yaml
```

Kubernetes 会分配给 LoadBalancer Service 一个内部的虚拟 IP ，并暴露 NodePort。进一步的，Kubernetes 请求底层云平台创建一个负载均衡其，作为访问 LoadBalancer Service 的外部访问入口。 负载均衡器由底层云平台创建提供，会包含一个 LoadBalancerIP，可以认为是 LoadBalancer Service 的外部 IP，查询 LoadBalancer Service：

```
$ kubectl get svc my-nginx 
```

其中  EXTERNAL-IP：xx 就是  LoadBalancer Service 的外部 IP，负载均衡器将每一个 Node 作为后端，当请求 xxx 时，负载均衡器将转发请求到相应的 [NodeIP]:[NodePort] ，从而访问到  LoadBalancer Service 。

## Ingress

Kubernetes 提供了一种 HTTP 方式的路由转发机制，称为 Ingress 。Ingress 的实现需要两个组件支持，Ingress Contriller 和 HTTP 代理服务器。HTTP 代理服务器将会转发外部的 HTTP 请求到 Service，而 Ingress Controller 则需要监控 Kubernetes API，实时更新 HTTP代理服务器的转发规则。
创建一个 Ingress ，

``` yaml my-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: my.examole.com
    http:
      paths:
      - path: /app
      backend:
        serviceName: my-app
        servicePort: 80
```

Ingress 定义中的 .spec.rules 设置了转发规则，启用配置了一条规则，当 HTTP 请求的 host 为 my.example.com 且 path 为/app 时，转发到 Service my-app 的 80 端口。

通过定义文件创建 Ingress:

```
$ kubectl create -f my-ingress.yaml
```

当 Ingress 创建成功后，需要 Ingress Controller 根据 Ingress 的配置，设置 HTTP 代理服务器的转发策略，外部通过 HTTP 代理服务器就可以访问到 Service。

Ingeress Controller 和 HTTP 代理服务器是作为外部组件运行的，官方提佛那个了 GCE Load-Balance 作为 HTTP 代理服务器，另外也可以使用 HAPorxy 或者 Nginx 方案。

