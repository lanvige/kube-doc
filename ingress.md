# Ingress

L7 层的转发，Service 对外的方式之一，另几种分别是

* Load Balance
* NodePort
* Ingress


## Ingress Controller

Ingress 正常工作需要集群中运行 Ingress Controller。Ingress Controller 与其他作为 kube-controller-manager 中的在集群创建时自动启动的 controller 成员不同，需要用户选择最适合自己集群的 Ingress Controller，或者自己实现一个。

Ingress Controller 以 Kubernetes Pod 的方式部署，以 daemon 方式运行，保持 watch Apiserver 的 /ingress 接口以更新 Ingress 资源，以满足 Ingress 的请求.

* traefik ingress 提供了一个 Traefik Ingress Controller 的实践案例
* kubernetes/ingress-nginx 提供了一个详细的 Nginx Ingress Controller 示例
* kubernetes/ingress-gce 提供了一个用于 GCE 的 Ingress Controller 示例


## TLS 

## 




## REF::

https://mritd.me/2017/03/04/how-to-use-nginx-ingress/


