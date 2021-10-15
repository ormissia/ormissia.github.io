---
title: Linux部署Kubernetes流程
date: 2021-10-14T18:09:43+08:00
hero: head.svg
menu:
  sidebar:
    name: Linux部署Kubernetes流程
    identifier: deployment-linux-kubernetes
    parent: deployment
    weight: 3007
---

## 初始化环境

### 防火墙

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
```

### SELinux

```shell
vi /etc/selinux/config
```

```shell
SELINUX=disabled
```

### Swap

```shell
vi /etc/fstab
```

注释掉`swap`这一行

```shell
/.swapfile	none	swap	sw,comment=cloudconfig	0	0
```

重启之后查看关闭是否成功

```shell
free -m
```

显示如下内容，`swap`关闭成功

```shell
              total        used        free      shared  buff/cache   available
Mem:          23114         402       22299          32         411       20597
Swap:             0           0           0
```

### ulimit

```shell
echo "ulimit -n 65535" >> /etc/profile
echo "*	hard	nofile	65535" >> /etc/security/limits.conf
```

重启之后检查是否配置成功

```shell
ulimit -n
```

## SSH免密（非必须）

执行命令，一路回车，即可获得当前节点的公钥

```shell
ssh-keygen -t rsa
cat id_rsa.pub
```

## iptables

查看`br_netfilter`模块是否开启

```shell
lsmod | grep br_netfilter
```

如果没有看到输出则执行命令开启

```shell
modprobe br_netfilter
```

作为Linux节点的iptables正确查看桥接流量的要求，应该确保`net.bridge.bridge-nf-call-iptables`在`sysctl`配置中设置为`1`

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

## 安装Docker

```shell
yum install -y yum-utils
```

```shell
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```shell
yum install docker-ce docker-ce-cli containerd.io -y
```

```shell
systemctl start docker
systemctl enable docker
```

修改Docker Cgroup Driver 为 systemd（大坑一）

```shell
vi /usr/lib/systemd/system/docker.service
```

修改配置文件中的启动参数

```shell
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
```

重启Docker

```shell
systemctl daemon-reload
systemctl restart docker
```

## 安装Kubernetes

- `kubeadm`：引导集群的命令。
- `kubelet`：在集群中的所有机器上运行的组件，并执行诸如启动`Pod`和容器之类的操作。
- `kubectl`：用于与集群通信的命令行实用程序

使用`yum`安装，添加`yum`源

```shell
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

安装并启动

```shell
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

至此，集群中所有节点都是相同操作

## 初始化master

使用`kubeadm`作为集群的初始化工具

```shell
kubeadm init --kubernetes-version=v1.22.2 --pod-network-cidr=10.244.0.0/16
```

成功初始化之后会出现提示，其中有三个需要手动执行，最后一个是需要在从节点上执行（此处有坑）

```shell
mkdir -p #HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf #HOME/.kube/config
sudo chown #(id -u):#(id -g) #HOME/.kube/config

kubeadm join --control-plane 10.0.0.105:6443 --token axqrzz.ouonxxxxxxgdvgjz \
	--discovery-token-ca-cert-hash sha256:3a57ff0d78b1f85xxxxxxxxxxxxxxxxxxxxe0f90f47037a31a33
```

使用`kubectl get nodes`命令查看节点

```shell
NAME         STATUS    ROLES                  AGE    VERSION
arm-node-1   NotReady  control-plane,master   1m     v1.22.2
```

此时可以看到`master`处于`NotReady`状态

查看集群中`pod`状态

> 这是后期补的，所以时间很久

```shell
kubectl get pod --all-namespaces

# 输出
NAMESPACE     NAME                                 READY   STATUS    RESTARTS       AGE
kube-system   coredns-78fcd69978-2vcqt             1/1     Pending   1 (126m ago)   12h
kube-system   coredns-78fcd69978-xg98g             1/1     Pending   1 (126m ago)   12h
kube-system   etcd-arm-node-1                      1/1     Running   2 (126m ago)   12h
kube-system   kube-apiserver-arm-node-1            1/1     Running   2 (126m ago)   12h
kube-system   kube-controller-manager-arm-node-1   1/1     Running   2 (126m ago)   12h
kube-system   kube-proxy-q5m5k                     1/1     Running   1 (126m ago)   12h
kube-system   kube-scheduler-arm-node-1            1/1     Running   2 (126m ago)   12h
```

可以看到两个`coredns`的`pod`处于`Pending`状态，这是由于缺少网络组件  
这里我们选择安装`flannel`

大坑二

```shell
kubctl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

执行这条命令之后查看pod状态会出现`ErrImagePull`的提示  
原因是`rancher/flannel-cni-plugin:v1.2`的镜像拉取不到  
到网上查了一番，看好多是相同的报错，但是因为`quay.io/coreos/flannel`的镜像拉不到，于是我手动拉了一下`rancher/flannel-cni-plugin`这个镜像

```shell
docker pull rancher/flannel-cni-plugin:v1.2
# 输出
Error response from daemon: pull access denied for rancher/flannel-cni-plugin, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

咦？找不到镜像？
于是我去Docker Hub的页面去搜了一下，居然没有这个镜像

![dockerhub](dockerhub.png)

~~真是啥坑都有~~

去[flannel-io/flannel的issue](https://github.com/flannel-io/flannel/issues/1482)中看了一下，最新的一条就是这个相同的问题

好吧，删掉换个旧版本重新来过

在GitHub的`flannel-io/flannel`仓库中切到`v0.14.1`版本中看了一下，没有使用到`rancher/flannel-cni-plugin`这个镜像

```shell
kubctl delete -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
kubctl apply -f https://raw.githubusercontent.com/flannel-io/flannel/release/v0.14.1/Documentation/kube-flannel.yml
```

这回成功了，重新查看`pod`和节点状态

```shell
kubectl get pod --all-namespaces

# 输出
NAMESPACE     NAME                                 READY   STATUS    RESTARTS       AGE
kube-system   coredns-78fcd69978-2vcqt             1/1     Running   1 (126m ago)   12h
kube-system   coredns-78fcd69978-xg98g             1/1     Running   1 (126m ago)   12h
kube-system   etcd-arm-node-1                      1/1     Running   2 (126m ago)   12h
kube-system   kube-apiserver-arm-node-1            1/1     Running   2 (126m ago)   12h
kube-system   kube-controller-manager-arm-node-1   1/1     Running   2 (126m ago)   12h
kube-system   kube-flannel-ds-kdpgp                1/1     Running   1 (126m ago)   3h32m
kube-system   kube-proxy-q5m5k                     1/1     Running   1 (126m ago)   12h
kube-system   kube-scheduler-arm-node-1            1/1     Running   2 (126m ago)   12h
```

```shell
kubectl get nodes

# 输出
NAME         STATUS   ROLES                  AGE    VERSION
arm-node-1   Ready    control-plane,master   13h    v1.22.2
```

## 初始化从节点（大坑三）

对于从节点，安装完`kubeadm`，`kubelet`，`kebectl`三个组件后，直接运行`master`节点`kubeadm init`完成之后的输出结果中的最后一条命令

```shell
kubeadm join --control-plane 10.0.0.105:6443 --token axqrzz.ouonxxxxxxgdvgjz \
	--discovery-token-ca-cert-hash sha256:3a57ff0d78b1f85xxxxxxxxxxxxxxxxxxxxe0f90f47037a31a33
```

但是这里我踩到一个坑，复制的这条命令粘贴到命令行中后，在第二行开头会莫名其妙多出来一个`.`

![kubeadmjoin](kubeadmjoin.png)

造成执行`kubeadm join`时一直报参数数量的错误

```shell
accepts at most 1 arg(s), received 3
To see the stack trace of this error execute with --v=5 or higher
```

_这个问题也查了好久，最后将命令改为一行去执行_

```shell
kubeadm join --control-plane 10.0.0.105:6443 --token axqrzz.ouonxxxxxxgdvgjz --discovery-token-ca-cert-hash sha256:3a57ff0d78b1f85xxxxxxxxxxxxxxxxxxxxe0f90f47037a31a33
```

命令执行返回加入集群成功后查看节点状态

```shell
kubectl get nodes

# 输出
NAME         STATUS   ROLES                  AGE    VERSION
arm-node-1   Ready    control-plane,master   13h    v1.22.2
arm-node-2   Ready    <none>                 170m   v1.22.2
```

可以看到从节点成功加入主节点




## 参考链接

- [k8s官网](https://kubernetes.io/)
- [kubectl开启shell自动补全](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion)
- [Docker 文档](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [flannel GitHub](https://github.com/flannel-io/flannel)
- [v0.14.1 kube-flannel.yml](https://raw.githubusercontent.com/flannel-io/flannel/release/v0.14.1/Documentation/kube-flannel.yml)

## v0.14.1/kube-flannel.yml

```yaml
# v0.14.1/kube-flannel.yml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.14.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.14.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
```
