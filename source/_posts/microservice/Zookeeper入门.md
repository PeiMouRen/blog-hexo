---
title: Zookeeper入门
date: 2023-04-04 10:14:36
categories:
  - 微服务
tags:
  - Zookeeper
---

# 简介

> ZooKeeper is a high-performance coordination service for distributed applications. It exposes common services - such as naming, configuration management, synchronization, and group services - in a simple interface so you don't have to write them from scratch. You can use it off-the-shelf to implement consensus, group management, leader election, and presence protocols. And you can build on it for your own, specific needs.

**Zookeeper**是一个微服务的注册中心，类似的产品还有**Nacos、Eureka**等。

官网：`https://zookeeper.apache.org/`

# 安装

> 安装Zookeeper集群环境

下载地址：`https://zookeeper.apache.org/releases.html`

环境要求：jdk

```bash
tar -zxvf apache-zookeeper-3.7.1-bin.tar.gz
cd apache-zookeeper-3.7.1-bin/
cp conf/zoo_sample.cfg conf/zoo.cfg
vi conf/zoo.cfg
```

**zoo.cfg**

> 注意：在一台主机上搭建伪集群时，dataDir、clientPort及server.x的两个端口号要不一致。

```
tickTime=2000
dataDir=/usr/local/zookeeper2181/data
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```

**myid**

在**dataDir**目录下新建**myid**文件，内容为该zookeeper在集群中的编号，**zoo.cfg**中**server.x**的**x**指的就是**myid**文件中的编号。集群环境下该编号要确保唯一。

```bash
echo 1 > /usr/local/zookeeper2181/data/myid
```

# 常用命令

## 服务端

```bash
bin/zkServer.sh start|start-foreground|stop|version|restart|status|print-cmd
```

## 客户端

```bash
bin/zkCli.sh -server localhost:2181
# help查看命令
```

# 常用配置

> 列举一些zoo.cfg中的配置

**tickTime**：zookeeper心跳时间，单位是毫秒。

**dataDir**：存放数据的目录。

**clientPort** ：监听客户端连接的端口。

**initLimit**：集群环境下从机连接和同步到主机的超时时间。配置为5表示5个**tickTime**。

**syncLimit**：集群环境下从机同步到主机的超时时间。配置为2表示2个**tickTime**。

**server.x=[hostname]:nnnnn[:nnnnn]**：组成集群环境的zookeeper主机，**x**表示每个zookeeper的**myid**文件中的内容。**hostname**表示主机名，第一个端口号是集群内通信使用，第二个端口号是选举leader使用。

# 可视化工具

prettyZoo
