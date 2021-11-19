---
title: HDFS基础知识
date: 2021-09-24T17:22:14+08:00
hero: head.svg
menu:
  sidebar:
    name: HDFS基础知识
    identifier: knowledge-hadoop-hdfs
    parent: knowledge
    weight: 2007
---

## HDFS架构

- HDFS是一个主从（Master/Slaves）架构
- 由一个NameNode和一些DataNode组成
- 面向文件包含：文件元数据（metadata）和文件数据（data）
- NameNode负责存储和管理文件元数据，并维护了一个层次型的文件目录树
- DataNode负责存储文件数据（block块），并提供block的读写
- DataNode与NameNode维持心跳，并汇报自己持有的block信息
- Client和NameNode交互文件元数据和DataNode交互文件block数据

目录树结构
角色即进程

## Hadoop集群中HDFS节点角色

### Master

### Standy
