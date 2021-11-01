---
title: Grafana部署流程
date: 2021-10-09T20:45:54+08:00
hero: head.png
menu:
  sidebar:
    name: Grafana部署流程
    identifier: deployment-linux-grafana
    parent: deployment
    weight: 3004
---

Grafana的安装比较简单，打开官网下载页面，选择对应的系统以及需要的版本号，按照指引执行命令即可。

## 程序下载

```shell
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.2.0-1.x86_64.rpm
sudo yum install grafana-enterprise-8.2.0-1.x86_64.rpm
```

## 启动程序

```shell
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

## 验证

Grafana默认端口为`:3000`，默认用户名密码均为`admin`，程序启动后即可通过3000端口访问管理页面。

![grafana-login](grafana-login.png)

## 开机自启

```shell
sudo systemctl enable grafana-server
```

## Kubernetes部署脚本

### Deployment&Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  labels:
    app: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - podAffinityTerm:
                topologyKey: kubernetes.io/hostname
                labelSelector:
                  matchExpressions:
                    - key: role
                      operator: In
                      values:
                        - data
              weight: 100
      containers:
        - name: grafana
          image: grafana/grafana-enterprise
          ports:
            - containerPort: 3000
          env:
            - name: GF_PATHS_CONFIG
              value: /etc/grafana/grafana.ini
            - name: GF_PATHS_DATA
              value: /var/lib/grafana
            - name: GF_PATHS_HOME
              value: /usr/share/grafana
            - name: GF_PATHS_LOGS
              value: /var/log/grafana
            - name: GF_PATHS_PLUGINS
              value: /var/lib/grafana/plugins
            - name: GF_PATHS_PROVISIONING
              value: /etc/grafana/provisioning
          volumeMounts:
            - mountPath: /etc/grafana/
              name: grafana-conf-dir
            - mountPath: /var/lib/grafana
              name: grafana-conf-dir
      volumes:
        - name: grafana-conf-dir
          persistentVolumeClaim:
            claimName: grafana-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  type: ClusterIP
  selector:
    app: grafana
  ports:
    - port: 3000
      targetPort: 3000
      protocol: TCP
```

### PV&PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: node1-grafana-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: node1-grafana-pv
  nfs:
    path: /data/nfs/kubernetes/grafana
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: node1-grafana-pv
```

### Ingress

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: grafana-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web

spec:
  rules:
    - host: grafana.ormissia.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: grafana
                port:
                  number: 3000
```

## 参考连接

- [Grafana官网](https://grafana.com/)
- [Grafana下载页面](https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1)
- [Grafana安装文档](https://grafana.com/docs/grafana/latest/installation/rpm/#2-start-the-server)
