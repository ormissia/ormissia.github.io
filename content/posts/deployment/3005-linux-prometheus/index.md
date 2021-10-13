---
title: Linux部署Prometheus流程
date: 2021-10-09T20:46:05+08:00
hero: head.png
menu:
  sidebar:
    name: Linux部署Prometheus流程
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

## 参考链接

[Prometheus官网](https://prometheus.io)
[Prometheus下载页面](https://prometheus.io/download/)
[Prometheus文档](https://prometheus.io/docs/introduction/overview/)
