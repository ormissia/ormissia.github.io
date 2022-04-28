---
title: 修改Docker镜像源
date: 2021-11-04T09:40:14+08:00
hero: head.svg
menu:
  sidebar:
    name: 修改Docker镜像源
    identifier: problems-docker-image-source
    parent: problems
    weight: 5005
---

---

#docker

---

## 问题现象

国内某些网络环境下，会出现`docker pull`无法拉取镜像的情况

## 解决办法

> 修改`Docker`镜像源

新建或修改

```shell
vi /etc/docker/daemon.json
```

```json
{
  "registry-mirrors": [
    "http://hub-mirror.c.163.com"
  ]
}
```

最后重启`Docker`

```shell
systemctl restart docker.service
```
