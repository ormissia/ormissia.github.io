---
title: k8s中通过Headless连接StatefulSet
date: 2021-09-24T17:22:14+08:00
hero: head.svg
menu:
  sidebar:
    name: k8s中通过Headless连接StatefulSet
    identifier: knowledge-kubernetes-handless-statefulset
    parent: knowledge
    weight: 2009
---

> 连接一些多实例的服务（比如Kafka、ES）时，通常是在client端做负载均衡。  
> 假如这种集群又恰好跑在k8s中，如果是普通业务类型的服务，通常是创建一个Service来做为一个代理去访问不同实例，从而达到负载均衡的目的。  
> 但是诸如如：Kafka、ES类型的服务，还用Service来做负载均衡，显然就不那么合理了（诚然，Kafka、ES这种东西多半是不会跑在k8s上的，这里只是作为一个引子，不在本文讨论的范畴）。

## 实验环境

- 多实例服务`whoami`在`kube-test-1`的命名空间下
- 多实例服务`whoami`以`StatefulSet`方式部署，设置为3个实例，会自动创建`whoami-0`、`whoami-1`以及`whoami-2`三个`Pod`
- 给`StatefulSet`创建`Headless`类型的`Service`
- 模拟客户端使用`Nginx`镜像，部署在`kube-test-2`的命名空间下（使用curl命令模拟）

*本实验创建资源使用的`k8s dashboard`，创建的资源默认放在选中的明明空间下，因此`yml`文件中未指定`namespace`。*

## Server cluster

> 服务端模拟相关资源在`kube-test-1`下创建

### StatefulSet

使用`traefik/whoami`镜像来模拟服务端

这里使用`StatefulSet`的方式创建服务端。`spec.replicas`设为3，此时会自动创建`whoami-0`、`whoami-1`以及`whoami-2`三个`Pod`。

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: whoami
  labels:
    app: whoami
spec:
  replicas: 3
  selector:
    matchLabels:
      app: whoami
  serviceName: whoami
  template:
    metadata:
      name: whoami
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami
          ports:
            - containerPort: 80
```

**注意这里的`spec.serviceName`必须与下面的`Service`名字相同，否则调用时候`pod`的`subdomain`只能使用`IP`**

```shell
$ k get pod -n kube-test-1 -o wide | grep whoami
whoami-0                 1/1     Running   0          29m   10.244.1.220   node2   <none>           <none>
whoami-1                 1/1     Running   0          29m   10.244.0.52    node1   <none>           <none>
whoami-2                 1/1     Running   0          29m   10.244.1.221   node2   <none>           <none>
```

### Headless

使用`Headless`的方式向外暴露`Pod`，不允许使用`Service`的负载均衡

```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  clusterIP: None
  ports:
    - port: 80
  selector:
    app: whoami
```

```shell
$ k get service -n kube-test-1 -o wide | grep whoami
whoami   ClusterIP   None         <none>        80/TCP    10s   app=whoami
```



## Client

> 客户端模拟相关资源在`kube-test-2`下创建

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
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
          image: nginx
```

`nginx`用来模拟客户端，调用镜像自带的curl命令

进入`nginx`容器

```shell
$ k exec -it nginx-6799fc88d8-cgvtp -n kube-test-2 /bin/bash
```

使用`curl`命令分别调用几个服务端

```shell
$ curl whoami-0.whoami.kube-test-1.svc.cluster.local
Hostname: whoami-0
IP: 127.0.0.1
IP: 10.244.1.220
RemoteAddr: 10.244.0.53:39400
GET / HTTP/1.1
Host: whoami-0.whoami.kube-test-1.svc.cluster.local
User-Agent: curl/7.64.0
Accept: */*

$ curl whoami-1.whoami.kube-test-1.svc.cluster.local
Hostname: whoami-1
IP: 127.0.0.1
IP: 10.244.0.52
RemoteAddr: 10.244.0.53:40354
GET / HTTP/1.1
Host: whoami-1.whoami.kube-test-1.svc.cluster.local
User-Agent: curl/7.64.0
Accept: */*

$ curl whoami-2.whoami.kube-test-1.svc.cluster.local
Hostname: whoami-2
IP: 127.0.0.1
IP: 10.244.1.221
RemoteAddr: 10.244.0.53:38370
GET / HTTP/1.1
Host: whoami-2.whoami.kube-test-1.svc.cluster.local
User-Agent: curl/7.64.0
Accept: */*
```

可以看到分别访问到三个不同的服务端`pod`了。

## 总结

访问的url格式为：

```
pod-ip-address.deployment-name.my-namespace.svc.cluster-domain.example
```

## 参考链接

- [k8s 文档：A/AAAA 记录](https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/#a-aaaa-%E8%AE%B0%E5%BD%95-1)

