---
title: Redis默认配置文件修改
date: 2022-01-13T15:37:06+08:00
hero: head.svg
menu:
  sidebar:
    name: Redis默认配置文件修改
    identifier: deployment-linux-redis
    parent: deployment
    weight: 3008
---

> 与MySQL类似，Redis安装完也不能直接使用，默认的配置文件有几处需要修改

使用`yum`安装的Redis默认配置文件路径：`/etc/redis.conf`

## 允许访问地址

```shell
# bind 127.0.0.1
```

将只限本地访问的配置注释掉

## 修改保护模式

```shell
protected-mode no 
```

将保护模式修改为`no`

## 启用守护进程

```shell
daemonize yes
```

将守护进程设置为`yes`
