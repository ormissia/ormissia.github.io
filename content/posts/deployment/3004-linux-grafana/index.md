---
title: Linux部署Grafana流程
date: 2021-10-09T20:45:54+08:00
hero: head.png
menu:
  sidebar:
    name: Linux部署Grafana流程
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


## 开机自启

```shell
sudo systemctl enable grafana-server
```

## 参考连接

- [Grafana官网](https://grafana.com/)
- [Grafana下载页面](https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1)
- [Grafana安装文档](https://grafana.com/docs/grafana/latest/installation/rpm/#2-start-the-server)
