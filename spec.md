# Spec

声明式的格式

我们操作 Kubernetes 时，有多种方式，像 CLI 式的，


## CLI

kubectl 后面跟上一长串命令就可以了。


```
kubectl run hello-node --image=hello-node:v1 --port=8080
```

这种方式的缺点就是命令行可能会很长，而且不方便复用。


## SPEC

SPEC 就是把 CLI 的参数写到一个文件中，然后把文件作为参数来启动。通常我们用 yaml 格式来进行描写，当然也可以用 json。


###  Pod

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

启动

```
$ kubectl create -f pod-nginx.yaml
```


### Deployment

示例：

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
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

启动命令

```
$ kubectl apply -f nginx-deployment.yaml
```


