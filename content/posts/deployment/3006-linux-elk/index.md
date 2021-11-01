---
title: ELK部署流程
date: 2021-10-13T15:28:13+08:00
hero: head.svg
menu:
  sidebar:
    name: ELK部署流程
    identifier: deployment-linux-elk
    parent: deployment
    weight: 3006
---

## Elasticsearch

### Deployment&Service

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app: elasticsearch
    role: master
  name: elasticsearch-master
spec:
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: elasticsearch
      role: master
  serviceName: es-master
  template:
    metadata:
      labels:
        app: elasticsearch
        role: master
    spec:
      serviceAccountName: elasticsearch-admin
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - topologyKey: kubernetes.io/hostname
              labelSelector:
                matchExpressions:
                  - key: role
                    operator: In
                    values:
                      - master
      containers:
        - name: elasticsearch-master
          image: elasticsearch:7.14.2
          lifecycle:
            postStart:
              exec:
                command: [ "/bin/bash", "-c", "sysctl -w vm.max_map_count=262144; ulimit -l unlimited; chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/data;" ]
          ports:
            - containerPort: 9200
              protocol: TCP
            - containerPort: 9300
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 1Gi
            requests:
              cpu: 10m
              memory: 512Mi
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
                  apiVersion: v1
            - name: path.data
              value: /usr/share/elasticsearch/data/${MY_POD_NAME}
            - name: cluster.name
              value: elasticsearch-k8s-cluster
            - name: discovery.seed_hosts
              value: elasticsearch-discovery
            - name: node.master
              value: "true"
            - name: node.data
              value: "false"
            - name: node.ingest
              value: "false"
            - name: ES_JAVA_OPTS
              value: "-Xms512m -Xmx512m"
            - name: cluster.initial_master_nodes
              value: elasticsearch-master-0,elasticsearch-master-1,elasticsearch-master-2
#            - name: xpack.security.enabled
#              value: "true"
#            - name: xpack.security.transport.ssl.enabled
#              value: "true"
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /usr/share/elasticsearch/data
              name: es-master-pvc
            - mountPath: /usr/share/elasticsearch/plugins
              name: es-plugins-pvc
      volumes:
        - name: es-master-pvc
          persistentVolumeClaim:
            claimName: es-master-pvc
        - name: es-plugins-pvc
          persistentVolumeClaim:
            claimName: es-plugins-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-discovery
spec:
  ports:
    - port: 9300
  selector:
    app: elasticsearch

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app: elasticsearch
    role: data
  name: elasticsearch-data
spec:
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: elasticsearch
      role: data
  serviceName: es-data
  template:
    metadata:
      labels:
        app: elasticsearch
        role: data
    spec:
      serviceAccountName: elasticsearch-admin
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
        - name: elasticsearch-data
          image: elasticsearch:7.14.2
          lifecycle:
            postStart:
              exec:
                command: [ "/bin/bash", "-c", "sysctl -w vm.max_map_count=262144; ulimit -l unlimited; chown -R elasticsearch:elasticsearch /usr/share/elasticsearch;" ]
          ports:
            - containerPort: 9200
              protocol: TCP
            - containerPort: 9300
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 1Gi
            requests:
              cpu: 10m
              memory: 512Mi
          env:
            - name: MY_POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
                  apiVersion: v1
            - name: path.data
              value: /usr/share/elasticsearch/data/${MY_POD_NAME}
            - name: cluster.name
              value: elasticsearch-k8s-cluster
            - name: discovery.seed_hosts
              value: elasticsearch-discovery
            - name: node.master
              value: "false"
            - name: node.data
              value: "true"
            - name: ES_JAVA_OPTS
              value: "-Xms300m -Xmx300m"
            - name: cluster.initial_master_nodes
              value: elasticsearch-master-0,elasticsearch-master-1,elasticsearch-master-2
#            - name: xpack.security.enabled
#              value: "true"
#            - name: xpack.security.transport.ssl.enabled
#              value: "true"
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /usr/share/elasticsearch/data
              name: es-data-pvc
            - mountPath: /usr/share/elasticsearch/plugins
              name: es-plugins-pvc
      volumes:
        - name: es-data-pvc
          persistentVolumeClaim:
            claimName: es-data-pvc
        - name: es-plugins-pvc
          persistentVolumeClaim:
            claimName: es-plugins-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
spec:
  type: ClusterIP
  ports:
    - port: 9200
      targetPort: 9200
      protocol: TCP
      name: http
  selector:
    app: elasticsearch
    role: data
```

### RBAC

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: elasticsearch-admin

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: elasticearch-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: elasticsearch-admin
    namespace: kube-elastic
```


### PV&PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: es-master-pv
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: es-master-pv
  nfs:
    path: /data/nfs/kubernetes/elastic/elasticsearch/master
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: es-master-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 512Mi
  storageClassName: es-master-pv

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: es-data-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: es-data-pv
  nfs:
    path: /data/nfs/kubernetes/elastic/elasticsearch/data
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: es-data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: es-data-pv

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: es-plugins-pv
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: es-plugins-pv
  nfs:
    path: /data/nfs/kubernetes/elastic/elasticsearch/plugins
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: es-plugins-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 512Mi
  storageClassName: es-plugins-pv
```

### Ingress

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: elasticsearch-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web

spec:
  rules:
    - host: elasticsearch.ormissia.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: elasticsearch
                port:
                  number: 9200
```



## Kibana

### Deployment&Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kibana
  name: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
        - name: kibana
          image: kibana:7.14.2
          ports:
            - containerPort: 5601
              protocol: TCP
          resources:
            limits:
              cpu: 100m
              memory: 600Gi
            requests:
              cpu: 10m
              memory: 400Mi
          env:
            - name: ELASTICSEARCH_URL
              value: http://elasticsearch:9200
            - name: I18N_LOCALE
              value: zh-CN
            - name: SERVER_PUBLICBASEURL
              value: https://kibana.ormissia.com

---
apiVersion: v1
kind: Service
metadata:
  name: kibana
spec:
  type: ClusterIP
  ports:
    - port: 5601
      targetPort: 5601
      protocol: TCP
  selector:
    app: kibana
```

### PV&PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kibana-pv
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: kibana-pv
  nfs:
    path: /data/nfs/kubernetes/elastic/kibana
    server: node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kibana-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 512Mi
  storageClassName: kibana-pv
```

### Ingress

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: kibana-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web

spec:
  rules:
    - host: kibana.ormissia.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: kibana
                port:
                  number: 5601
```
