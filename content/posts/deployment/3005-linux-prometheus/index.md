---
title: Prometheus部署流程
date: 2021-10-09T20:46:05+08:00
hero: head.png
menu:
  sidebar:
    name: Prometheus部署流程
    identifier: deployment-linux-prometheus
    parent: deployment
    weight: 3005
---

## 程序下载

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.30.3/prometheus-2.30.3.linux-amd64.tar.gz
```

## 解压并移动

```shell
tar -zxvf prometheus-2.30.3.linux-amd64.tar.gz
mv prometheus-2.30.3.linux-amd64 /usr/local/prometheus
```

## 添加到系统服务

### Unit配置文件

```shell
vi /usr/lib/systemd/system/prometheus.service
```

```shell
[Unit]
Description=Prometheus
Documentation=https://prometheus.io

[Service]
Type=simple
ExecStart=/usr/local/prometheus/prometheus \
--config.file=/usr/local/prometheus/prometheus.yml \
--storage.tsdb.path=/usr/local/prometheus/data
Restart=on-failure
WatchdogSec=10s

[Install]
WantedBy=multi-user.target
```

### 启动程序

```shell
sudo systemctl daemon-reload
sudo systemctl start prometheus.service
sudo systemctl status prometheus.service
```

### 开机自启

```shell
sudo systemctl enable prometheus.service
```

## 简单使用

Prometheus默认端口是9090，程序启动之后从浏览器访问页面。

输入以下表达式来绘制在自抓取Prometheus中发生的每秒HTTP请求率返回状态代码`200`的图表：

```sql
rate(promhttp_metric_handler_requests_total{code="200"}[1m])
```

## 配置文件

### 重新加载

```shell
curl -X POST http://127.0.0.1:9090/-/reload
```

## Kubernetes部署脚本

### Deployment&Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      initContainers:
        - name: prometheus-data-permission-setup
          image: busybox
          command: ["/bin/chmod","-R","777", "/data"]
          volumeMounts:
            - name: prometheus-data
              mountPath: /data
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - '--storage.tsdb.path=/prometheus'
            - '--config.file=/etc/prometheus/prometheus.yml'
          command:
            - /bin/prometheus
          ports:
            - name: web
              containerPort: 9090
          resources:
            limits:
              cpu: 20m
              memory: 150Mi
            requests:
              cpu: 5m
              memory: 80Mi
          volumeMounts:
            - name: config-volume
              mountPath: /etc/prometheus
            - name: prometheus-data
              mountPath: /prometheus
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
      serviceAccountName: prometheus
      volumes:
        - name: config-volume
          configMap:
            name: prometheus-config
        - name: prometheus-data
          persistentVolumeClaim:
            claimName: prometheus-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  type: ClusterIP
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
      protocol: TCP
```

### RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
      - nodes/proxy
      - services
      - endpoints
      - pods
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - nonResourceURLs:
      - /metrics
    verbs:
      - get

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: kube-basic
```

### PV&PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: node1-prometheus-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: node1-prometheus-pv
  nfs:
    path: /data/nfs/kubernetes/prometheus
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: node1-prometheus-pv
```

### Ingress

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: prometheus-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web

spec:
  rules:
    - host: prometheus.ormissia.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: prometheus
                port:
                  number: 9090
```

### Node Exporter Deployment&Service

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostPID: true
      hostIPC: true
      hostNetwork: true
      tolerations:
        - key:  node-role.kubernetes.io/master
          operator: Exists
          effect: NoSchedule
      containers:
        - name: node-exporter
          image: prom/node-exporter
          ports:
            - containerPort: 9100
          resources:
            limits:
              cpu: 10m
              memory: 30Mi
            requests:
              cpu: 2m
              memory: 10Mi
          securityContext:
            privileged: true
          args:
            - --path.procfs
            - /host/proc
            - --path.sysfs
            - /host/sys
            - --path.rootfs
            - /host/root
            - --collector.filesystem.ignored-mount-points
            - ^/(sys|proc|dev|host|etc)($|/)
            - --collector.processes
          volumeMounts:
            - mountPath: /host/dev
              name: dev
            - mountPath: /host/proc
              name: proc
            - mountPath: /host/sys
              name: sys
            - mountPath: /host/root
              name: rootfs
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: dev
          hostPath:
            path: /dev
        - name: sys
          hostPath:
            path: /sys
        - name: rootfs
          hostPath:
            path: /
```

### Prometheus Config

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node1-exporter"
    static_configs:
      - targets: ["gateway-node-exporter:9100"]
    relabel_configs:
      - source_labels: [__address__]
        regex: (.*)
        replacement: gateway
        target_label: instance
        action: replace
  - job_name: 'kubernetes-node-exporter'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:9100'
        target_label: __address__
        action: replace
      - source_labels: [__meta_kubernetes_node_name]
        action: replace
        target_label: node
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - source_labels: [__meta_kubernetes_node_address_InternalIP]
        action: replace
        target_label: ip
```

## 参考链接

[Prometheus官网](https://prometheus.io)
[Prometheus下载页面](https://prometheus.io/download/)
[Prometheus文档](https://prometheus.io/docs/introduction/overview/)
