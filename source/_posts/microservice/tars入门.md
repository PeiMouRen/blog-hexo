---
title: tars入门
date: 2022-09-08 09:14:33
categories:
  - 微服务
tags:
  - tars
---

[toc]

# 简介

> TARS 是 Linux 基金会的开源项目，它是基于名字服务使用 TARS 协议的高性能 RPC 开发框架，配套一体化的运营管理平台，并通过伸缩调度，实现运维半托管服务。
>
> TARS 是腾讯从 2008 年到今天一直在使用的后台逻辑层的统一应用框架，覆盖腾讯 100 多个产品。目前支持 C++,Java,PHP,Nodejs,Go 语言。该框架为用户提供了涉及到开发、运维、以及测试的一整套解决方案，帮助一个产品或者服务快速开发、部署、测试、上线。 它集可扩展协议编解码、高性能 RPC 通信框架、名字路由与发现、发布监控、日志统计、配置管理等于一体，通过它可以快速用微服务的方式构建自己的稳定可靠的分布式应用，并实现完整有效的服务治理。
>
> 目前该框架在腾讯内部，各大核心业务都在使用，颇受欢迎，基于该框架部署运行的服务节点规模达到上万个。

官方文档：`https://doc.tarsyun.com/`

# 部署

> 使用Docker部署

## 安装mysql

> Tars 框架的数据最终都存储在 mysql 中, 因此需要安装 mysql。

```bash
docker network create tars
docker pull mysql:5.7
docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 --network tars \
--privileged=true -v /tmp/mysql/log:/var/log/mysql -v /tmp/mysql/data:/var/lib/mysql \
-v /tmp/mysql/conf:/etc/mysql/conf.d mysql:5.7
# 查看mysql的ip(IPAddress属性)
docker inspect mysql01
```

## 安装框架服务

```bash
docker pull tarscloud/framework:latest
# mysql的配置换成自己的
# 3000端口为web程序端口
# 3001端口为web授权相关服务端口(docker>=v2.4.7可以不暴露该端口)
# 安装完毕后, 访问 http://${your_machine_ip}:3000 打开 web 管理平台
docker run -d \
    --name=tars-framework \
    --network=tars \
    -e MYSQL_HOST="172.19.0.2" \
    -e MYSQL_ROOT_PASSWORD="root" \
    -e MYSQL_USER=root \
    -e MYSQL_PORT=3306 \
    -e REBUILD=false \
    -e INET=eth0 \
    -e SLAVE=false \
    -v /tmp/tars/framework:/data/tars \
    -p 3000:3000 \
    -p 3001:3001 \
    tarscloud/framework:latest
```

## 安装应用节点

```bash
docker pull tarscloud/tars-node:latest
docker run -d \
    --name=tars-node \
    --network=tars \
    -e INET=eth0 \
    -e WEB_HOST="http://172.19.0.3:3000" \
    -v /tmp/tars:/data/tars \
    -p 9000-9010:9000-9010 \
    tarscloud/tars-node:latest
```

