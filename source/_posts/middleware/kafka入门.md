---
title: kafka入门
date: 2022-08-16 14:22:30
categories: 
 - 中间件
 - 消息队列
tags:
 - kafka
 - MQ
---

# 1. 搭建kafka单机环境

## 1.1. 版本

```
Linux:CentOS7
JDK:1.8
zookeeper:3.7.0
kafka:3.0.0
```

## 1.2. 安装环境

_Linux服务器上已有JDK环境和Zookeeper环境。_

## 1.3. 资源下载

```
# kafka官方下载
https://kafka.apache.org/downloads
```

## 1.4. 搭建步骤

```
# 将下载好的安装包放到/usr/local/kafka目录下，并解压
tar -zxvf kafka_2.12-3.0.0.tgz
# 编辑config/server.propoties文件
vi config/server.propoties
# 配置日志目录和zookeeper地址
log.dirs=/usr/local/kafka/datas
zookeeper.connect=node1:2181,node2:2181,node3:2181/kafka
# 添加环境变量
vi /etc/profile
# 追加kafka的配置
export KAFKA_HOME=/usr/local/kafka
export PATH=$PATH:$KAFKA_HOME/bin
# 设置新增的环境变量生效
source /etc/profile
```

## 1.5. 测试搭建结果

```
# 启动kafka
kafka-server-start.sh -daemon config/server.properties
# 创建主题（名称为kafkaTestTopic，1个分区，1个副本，这个node1为hostname）
kafka-topics.sh --bootstrap-server node1:9092 --create --topic kafkaTestTopic --partitions 1 --replication-factor 1
```

![创建kafka主题](创建kafka主题.png)

```
# 开启生产者服务，向kafkaTestTopic发送数据
kafka-console-producer.sh --bootstrap-server node1:9092 --topic kafkaTestTopic
# 重新打开一个终端，开启一个消费者服务
kafka-console-consumer.sh --bootstrap-server node1:9092 --topic kafkaTestTopic --from-beginning
# 在生产者控制台输入消息，观测到消费者控制台可接收到数据
```

![生产者发送消息](生产者发送消息.png)
![消费者收到消息](消费者收到消息.png)

# 2. 搭建kafka集群环境

## 2.1. 环境

_三台搭建好kafka单机环境的服务器，kafka单机环境搭建见上文。_

## 2.2. 搭建步骤

_与单机环境搭建最大的不同，就是三个节点要分别指定自己的`broker.id`。_

```
# 修改三个节点的broker.id分别为0、1、2
vi config/server.propoties
```

## 2.3. 测试搭建效果

*测试过程与单机环境测试步骤大致相同，稍有不同的地方为集群测试的`--bootstrap-server`可指定多个节点，中间用`,`隔开，且创建主题时`--partitions`参数和`--replication-factor`参数可根据集群数量灵活指定。*
