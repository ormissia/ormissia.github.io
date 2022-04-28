---
title: k8s dashboard token过期时间太短
date: 2021-11-01T10:37:18+08:00
hero: head.svg
menu:
  sidebar:
    name: k8s dashboard token过期时间太短
    identifier: problems-kubernetes-dashboard-token
    parent: problems
    weight: 5004
---

---

#kubernetes #k8s #token #dashboard

---

## 问题现象

kubernetes的dashboard登录token过期时间太短，不操作没一会就需要重新登录

![login](login.png)

## 解决办法

修改`kubernetes-dashboard`的`deployment`，加入一条`arg`参数：

```
- '--token-ttl=10800'
```

![deployment](deployment.png)
