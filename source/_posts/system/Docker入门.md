---
title: Docker入门
date: 2022-08-10 09:16:25
categories: 容器
tags:
 - Docker
---

[TOC]

# 什么是docker

虚拟容器技术，通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户APP及其运行环境能够做到**一次镜像，处处运行**。

三要素：镜像、容器、仓库

# 安装

安装步骤参考官网`https://docs.docker.com/engine/install/centos/`

## 环境

`CentOS7`

## 安装gcc

```bash
yum -y install gcc gcc-c++
```

## 设置存储库

```bash
# yum-utils包提供yum-config-manager程序
yum install -y yum-utils
# 设置稳定仓库，这里选择阿里云的仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装docker引擎

```bash
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 启动docker

```bash
systemctl start docker
# 查看docker进程
ps -ef | grep docker
# 查看docker版本
docker version
```

## hello world

```bash
# 运行docker自带的hello world
docker run hello-world
```

## 配置阿里云镜像加速

参考阿里云官网**容器镜像服务**`https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors`

# docker常用命令

## 帮助启动类命令

```bash
# 启动docker
systemctl start docker
# 停止docker
systemctl stop docker
# 重启docker
systemctl restart docker
# 查看docker状态
systemctl status docker
# 开机启动
systemctl enable docker
# 查看docker概要信息
docker info
# 查看docker帮助文档
docker --help
# 查看docker命令帮助文档
docker 具体命令 --help
```

## 镜像命令

```bash
# 列出本地所有镜像
docker images
# 从仓库查找镜像
docker search --limit 5 imageName # --limit表示只取前5条数据
# 从仓库拉取镜像
docker pull imageName:tag # :tag表示版本，不指定默认为lasted版本
# 查看镜像/容器/数据卷所占的空间
docker system df
# 删除镜像
docker rmi -f imageName/imageId # -f表示强制删除
docker rmi -f imageName:tag imageName:tag # 删除多个镜像
docker rmi -f $(docker images -qa) # 强制删除全部镜像
```

### 推送镜像到阿里云仓库

参考阿里云**容器镜像服务**

### 推送镜像到私有库

```bash
# 拉取docker registry镜像
docker pull registry
# 运行registry镜像
# -d表示后台运行 -p指定docker端口和容器端口 --name指定容器名称 -v指定数据卷 --privileged指定特权
docker run -d -p 5000:5000 --name registry -v 宿主机绝对路径:容器内绝对路径 --privileged=true registry:latest
# 修改镜像名，使之符合registry规范
docker image tag 镜像名:tag 192.168.5.110:5000/myfirstimage
# 修改配置文件，使docker支持http方式推送镜像
# 添加json串"insecure-registries":["192.168.5.110:5000"]
vi /etc/docker/daemon.json
# 重启docker服务及regisry容器
systemctl restart docker
docker start registry
# 推送镜像到registry
docker push 192.168.5.110:5000/myfirstimage
# 查看registry中的镜像
crul -XGET http://192.168.5.110:5000/v2/_catalog
# 拉取registry中的镜像
docker pull 192.168.5.110:5000/myfirstimage
```

## 容器命令

### 新建并启动容器

`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

常用options：

- --name：指定容器名称
- -d：后台运行容器并返回容器id，即启动**守护式容器**
- -i：以交互模式运行容器，通常与`-t`同时使用
- -t：为容器重新分配一个伪输入终端，即启动**交互式容器**，通常与`-i`同时使用
- -P：随机端口映射，大写P
- -p：指定端口映射，小写p

```bash
# 使用镜像centos:latest以交互模式启动一个名为myCont的容器，在容器内执行/bin/bash命令，使用exit退出终端
docker run -it --name=myCont centos /bin/bash
# 前台交互式启动redis，使用redis-cli -p 6379进入命令行
docker run -it redis
# 后台守护式启动redis
## 注意，有一些镜像是不能后台启动的，因为docker规定必须有一个前台进程，否则会自动退出
docker run -d redis
```

### 其他命令

```bash
# 查看正在运行的容器
docker ps
# 退出容器
exit # run进去容器，exit退出，容器停止
ctrl + p + q # run进去容器，ctrl + p + q退出，容器不停止
# 启动已经停止的容器
docker start 容器id/容器名
# 重启容器
docker restart 容器id/容器名
# 停止容器
docker stop 容器id/容器名
docker stop $(docker ps -aq) # 停止所有容器
# 强制停止容器
docker kill 容器id/容器名
# 删除已停止的容器
docker rm 容器id/容器名
# 查看容器日志
docker logs 容器id/容器名
# 查看容器内运行的进程
docker top 容器id/容器名
# 查看容器内部细节
docker inspect 容器id/容器名
# 进入正在运行的容器并以命令行交互
docker exec -it 容器id/容器名 /bin/bash # 新建进程，exit退出时不会停止容器
docker attach 容器id/容器名 # 不新建进程，exit退出时会停止容器
# 容器和主机上的文件互相拷贝
docker cp 容器id/容器名:容器内路径 主机路径 # 拷贝容器文件到主机
docker cp 主机路径 容器id/容器名:容器内路径 # 拷贝主机文件到容器
# 导入和导出容器
docker export 容器id/容器名 -o 文件名.tar
docker export 容器id/容器名 > 文件名.tar
docker import 文件名.tar 镜像用户/镜像名:镜像版本号
cat 文件名.tar | docker import  - 镜像用户/镜像名:镜像版本号
# 将容器提交为镜像
docker commit -m="提交的描述信息" -a="作者" 容器id/容器名 镜像用户/镜像名:镜像版本号
```

# docker volumes

映射宿主机和docker容器，将docker容器内的数据保存到宿主机的磁盘中，达到重要数据备份的目的。

```bash
# 启动容器时指定数据卷，宿主机目录和容器目录会自动创建，--privileged给容器root权限
# :rw表示允许读写，默认为该方式，可省略	:ro表示只读，即容器只可读取宿主机挂载目录下的文件
docker run -it -v 宿主机绝对路径:容器内绝对路径:rw --privileged=true 镜像名:tag
# 查看挂载情况查看Mounts参数
docker inspect 容器id
# 继承数据卷
docker run -it --volumes-from 容器id/容器名 镜像名:tag
```

# docker安装单机软件

## 步骤

搜索镜像(可在docker hub上搜索)、拉取镜像、查看镜像、启动镜像、停止容器、移除容器

## 安装tomcat

```bash
# 搜索镜像
docker search tomcat
# 拉取镜像
docker pull tomcat
# 查看镜像
docker images
# 启动镜像
docker run -d -p 8080:8080 --name t1 tomcat
# 访问tomcat主页，404问题解决：
# 1、检查防火墙和端口
# 2、如果是新版本的tomcat，需要进入到tomcat容器中，删除webapps目录，并移动webapps.dist到webapps
# 3、下载旧版tomcat镜像，如billygoo/tomcat8-jdk8 
# 通过http://192.168.5.110:8080/访问tomcat主页
```

## 安装mysql

```bash
# 搜索镜像
docker search mysql:5.7
# 拉取镜像
docker pull mysql:5.7
# 查看镜像
docker images
# 启动镜像
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 --privileged=true -v /tmp/mysql/log:/var/log/mysql -v /tmp/mysql/data:/var/lib/mysql -v /tmp/mysql/conf:/etc/mysql/conf.d mysql:5.7
# 新建my.cnf，通过数据卷同步给mysql容器
# 在my.cnf上输入
# [client] 
# default_character_set=utf8 
# [mysqld] 
# collation_server = utf8_general_ci 
# character_set_server = utf8
cd /tmp/mysql/conf
vi my.cnf
# 重启mysql容器，进入到mysql查看编码
docker restart 容器id
docker exec -it 容器id /bin/bash
mysql -uroot -p
show variables like 'character%';
# 进入容器
docker exec -it 容器id /bin/bash
# 进入mysql
mysql -uroot -p
```

## 安装redis

```bash
# 搜索镜像
docker search redis
# 拉取镜像
docker pull redis
# 查看镜像
docker images
# 新建redis.conf，从其他地方拷贝一份，放到/tmp/redis目录下，并修改如下配置
# 1.注释掉bind 127.0.0.1，允许redis外地连接
# 2.daemonize设置为no（默认），设置为yes会与docker run -d冲突
# 3.appendonly设置为yes，开启redis持久化
cat /tmp/redis/redis.conf
# 启动镜像,redis-server指定redis启动的配置文件
docker run --name r1 -d -p 6379:6379 --privileged=true -v /tmp/redis/redis.conf:/etc/redis/redis.conf -v /tmp/redis/data:/data  redis:latest redis-server /etc/redis/redis.conf
# 进入容器
docker exec -it 容器id /bin/bash
# 进入redis
redis-cli -p 6379
```

# docker安装集群软件

## mysql主从复制

```bash
# 1、新建主服务器容器实例3307
docker run \
--name mysql-master \
-e MYSQL_ROOT_PASSWORD=root \
-d -p 3307:3306 --privileged=true \
-v /tmp/mydata/mysql-master/log:/var/log/mysql \
-v /tmp/mydata/mysql-master/data:/var/lib/mysql \
-v /tmp/mydata/mysql-master/conf:/etc/mysql/conf.d \
mysql:5.7
# 2、在宿主机相应目录下配置mysql-master的my.cnf配置文件，文件内容见下文
cd /tmp/mydata/mysql-master/conf
vi my.cnf
# 3、重启并进入mysql-master容器
docker restart mysql-master
docker exec -it mysql-master /bin/bash
# 4、mysql-master内创建数据同步用户
mysql -uroot -p
create user 'slave'@'%' identified by '123456';
grant replication slave, replication client on *.* to 'slave'@'%';
# 5、新建从服务器容器实力3308(这里重开一个终端来操作)
docker run \
--name mysql-slave \
-e MYSQL_ROOT_PASSWORD=root \
-d -p 3308:3306 --privileged=true \
-v /tmp/mydata/mysql-slave/log:/var/log/mysql \
-v /tmp/mydata/mysql-slave/data:/var/lib/mysql \
-v /tmp/mydata/mysql-slave/conf:/etc/mysql/conf.d \
mysql:5.7
# 6、在宿主机相应目录下配置mysql-slave的my.cnf配置文件，文件内容见下文
cd /tmp/mydata/mysql-slave/conf
vi my.cnf
# 7、重启mysql-slave容器
docker restart mysql-slave
# 8、在mysql-master数据库中查看主从同步状态
show master status;
# 9、进入mysql-slave容器并连接mysql
docker exec -it mysql-slave /bin/bash
mysql -uroot -p
# 10、在mysql-slave数据库中配置主从复制
# master_host为宿主机ip，master_port为mysql-master的端口
# master_user和masterpassword为mysql-master上新建的同步用户
# master_log_file，指定mysql-slave要复制数据的日志文件，通过查看mysql-master的状态，获取file参数
# master_log_pos，指定mysql-slave从哪个位置开始复制数据，通过查看mysql-master的状态，获取position参# 数
# master_connect_retry为mysql-slave重试连接mysql-master的间隔时间，单位为秒
change master to master_host='192.168.5.110', master_user='slave',master_password='123456', 
master_port=3307, master_log_file='mysql_bin.000001', 
master_log_pos=617, master_connect_retry=30;
# 11、在mysql-slave数据库中查看主从同步状态
# \G表示以k-v键值对的方式显示
# 主要查看Slave_IO_Running和Slave_SQL_Running的值，为no表示还未开始复制
show slave status \G;
# 12、在mysql-slave数据库中开启主从复制
start slave;
# 13、再次在mysql-slave数据库中查看主从同步状态
# Slave_IO_Running和Slave_SQL_Running的值为yes
show slave status \G;
# 14、测试主从复制效果
# 在mysql-master数据库中新建库、新建表、插入数据
create database db1;
use db1;
create table t1(id int, name varchar(20));
insert into t1 values(1, 'zhangsan');
# 在mysql-slave数据库中查看记录
show databases;
use db1;
show tables;
select * from t1;
```

**mysql-master my.cnf配置文件内容**

```properties
[mysqld]
# 设置server_id，同一局域网中需要唯一
server_id=101
# 指定不需要同步的数据库名称
binlog_ignore_db=mysql
# 启用二进制日志
log_bin=mysql_bin
# 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
# 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
# 二进制日志过期清理时间，单位为天，默认为0，表示不自动清理
expire_logs_days=7
# 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
# 如：1062错误是指一些主键重复，1032错误是指主从数据库数据不一致
slave_skip_errors=1062
```

**mysql-slave my.cnf配置文件内容**

```properties
[mysqld]
# 设置server_id，同一局域网中需要唯一
server_id=102
# 指定不需要同步的数据库名称
binlog_ignore_db=mysql
# 启用二进制日志，以备slave作为其他数据库实例的master时使用
log_bin=mysql_bin
# 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
# 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
# 二进制日志过期清理时间，单位为天，默认为0，表示不自动清理
expire_logs_days=7
# 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
# 如：1062错误是指一些主键重复，1032错误是指主从数据库数据不一致
slave_skip_errors=1062
# relay_log配置中继日志
relay_log=mysql_relay_bin
# log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
# slave设置为只读（具有super权限的用户除外）
read_only=1
```

## redis集群

> redis集群采用hash slot的方式来分区，共有16384个slot，平均分给master节点即可

### 搭建3主3从集群

> 六个节点分别命名为redis-node1、redis-node2、redis-node3、redis-node4、redis-node5、redis-node6，端口为6381-6386

```bash
# 1、新建6个redis容器实例
# --net host表示使用宿主机的ip和端口（默认）
# --cluster-enabled表示开启redis集群
# --appendonly表示开启持久化
# --port指定redis端口号
# 启动node1
docker run -d --name redis-node1 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node1:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6381
# 启动node2
docker run -d --name redis-node2 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node2:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6382
# 启动node3
docker run -d --name redis-node3 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node3:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6383
# 启动node4
docker run -d --name redis-node4 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node4:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6384
# 启动node5
docker run -d --name redis-node5 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node5:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6385
# 启动node6
docker run -d --name redis-node6 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node6:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6386
# 2、进入redis-node1容器，为6个redis构建集群关系
docker exec -it redis-node1 /bin/bash
# 192.168.5.110为宿主机ip
# --cluster-replicas 1 表示为每个master节点创建一个slave节点
redis-cli --cluster create 192.168.5.110:6381 192.168.5.110:6382 \
192.168.5.110:6383 192.168.5.110:6384 192.168.5.110:6385 \
192.168.5.110:6386 --cluster-replicas 1
# 3、连接redis-node1节点，查看集群状态
# redis-cli --cluster check 192.168.5.110:6381 # 这种方式也可以查看集群状态
# 这里要注意，集群环境下，连接redis时，要加-c参数使用集群模式
redis-cli -p 6381
cluster info # 查看集群状态
cluster nodes # 查看集群节点
```

### 集群扩容

> 添加名为redis-node7端口为6387的主节点和名为redis-node8端口为6388的从节点，使集群扩容为4主4从

```bash
# 1、增加两个节点，
# 启动node7
docker run -d --name redis-node7 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node7:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6387
# 启动node8
docker run -d --name redis-node8 --net host --privileged=true \
-v /tmp/redis-cluster/redis-node8:/data redis:latest \
--cluster-enabled yes --appendonly yes --port 6388
# 2、进入redis-node7
docker exec -it redis-node7 /bin/bash
# 3、将新增的redis-node7（空槽号）作为master节点加入原集群，并检查集群状态
# 192.168.5.110:6381为原集群里的master节点，相当于redis-node7拜一拜redis-node1的码头
redis-cli --cluster add-node 192.168.5.110:6387 192.168.5.110:6381
redis-cli --cluster check 192.168.5.110:6381 # 此时集群中已经有了redis-node7节点，slot为0
# 4、重新分配slot并检查集群状态
# How many slots do you want to move (from 1 to 16384)? 输入4096 16384/master个数
# What is the receiving node ID? 输入redis-node7的id，查看集群状态能看到各节点的id
# Source node #1: 输入all
# Do you want to proceed with the proposed reshard plan (yes/no)? 输入yes
redis-cli --cluster reshard 192.168.5.110:6381
redis-cli --cluster check 192.168.5.110:6381 # 此时redis-node7节点已经有了4096个slot
# 5、给redis-node7添加从机redis-node8，并查看集群状态
# cluster-master-id指定redis-node7的id，通过集群状态获取
redis-cli --cluster add-node 192.168.5.110:6388 192.168.5.110:6387 \
--cluster-slave --cluster-master-id daa620710980195ec0279d65d10d87346ef94821
redis-cli --cluster check 192.168.5.110:6381 # 此时集群中已经有了redis-node8节点
```

### 集群缩容

> 剔除redis-node7、redis-node8两个节点，使集群缩容为3主3从

```bash
# 1、从集群中删除从机redis-node8，并查看集群状态
# d8e8cf55e9dda78543f2d7935eee1e0ada2fdd95为redis-node8的id
redis-cli --cluster del-node 192.168.5.110:6388 d8e8cf55e9dda78543f2d7935eee1e0ada2fdd95
redis-cli --cluster check 192.168.5.110:6381 # 此时集群中没有redis-node8节点了
# 2、清空redis-node7的slot，重新分配给其他master，之后查看集群状态
# How many slots do you want to move (from 1 to 16384)? 输入4096，即redis-node7的slot数
# What is the receiving node ID? 输入redis-node1的id，即将redis-node7的slot全部给redis-node1
# Source node #1: 输入redis-node7的id，即分配的是redis-node7的slot
# Source node #2: 输入done
# Do you want to proceed with the proposed reshard plan (yes/no)? 输入yes
redis-cli --cluster reshard 192.168.5.110:6381
redis-cli --cluster check 192.168.5.110:6381 # 此时redis-node7已经没有slot了
# 3、从集群中删除主机redis-node7，并查看集群状态
# daa620710980195ec0279d65d10d87346ef94821为redis-node7的id
redis-cli --cluster del-node 192.168.5.110:6387 daa620710980195ec0279d65d10d87346ef94821
redis-cli --cluster check 192.168.5.110:6381 # 此时集群中没有redis-node7节点了
# 4、redis-node7、redis-node8停机
docker stop redis-node7 redis-node8
```

# Dockerfile

> Dockerfile是用来构建docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。
>
> 构建三部曲：编写Dockerfile、docker build命令构建镜像、docker run运行容器
>
> 可参考tomcat的Dockerfile：`https://github.com/docker-library/tomcat/blob/master/10.0/jdk8/corretto/Dockerfile`
>
> 官方文档：`https://docs.docker.com/engine/reference/builder/`

## 基础知识

- 每条**保留字指令**都必须为**大写字母**，且后面至少要**跟随一个参数**
- 指定从上到下依次执行
- #表示注释
- 每条指令都会创建一个新的镜像，并对镜像进行提交

## 构建过程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器做出修改
3. 执行类似`docker commit`的操作提交一个新的镜像层
4. docker再基于上一步提交的镜像运行一个新容器
5. 执行`Dockerfile`中的下一条指令，直到所有的指令都运行完成

## 常用保留字指令

### FROM

基础镜像，描述当前镜像是基于哪个镜像的，指定一个已存在的镜像作为模板，放在Dockerfile的第一行

```
FROM amazoncorretto:8
```

### MAINTAINER

镜像维护者的姓名和邮箱

### RUN

容器构建时（即`docker build`）需要执行的命令，有`shell`和`exec`两种格式

- shell：RUN <命令行命令>

  ​			例如`RUN ./test.sh`，相当于在终端执行shell命令

- exec：RUN ["可执行文件", "参数1", "参数2"]

  ​			例如`RUN ["./test.sh", "dev", "offline"]`，等价于`RUN ./test.sh dev offline`

```
RUN mkdir /usr/local/tomcat
```

### EXPOSE

当前容器对外暴露的端口

```
EXPOSE 8080
```

### WORKDIR

容器创建后，终端登录成功后默认的工作目录

```
WORKDIR /usr/local/tomcat
```

### USER

指定镜像以哪个用户去执行，默认是root

### ENV

构建镜像过程中设置环境变量，后续可以用`$`引用该变量

```
ENV CATALINA_HOME /usr/local/tomcat
WORKDIR $CATALINA_HOME
```

### ADD

将宿主机目录下的文件拷贝到镜像，且会自动处理url和解压tar压缩包，相当于增强版的`ADD`指令

### COPY

类似`docker cp`，拷贝目录和文件到镜像中

### VOLUME

容器数据卷，用于持久化容器的数据，相当于`docker run -v`

### CMD

指定容器**启动后**要干的事，与`RUN`指令相似，也有`shell`和`exec`两种格式；

`CMD`指令可与`ENTRYPOINT`指令配合使用，在指定`ENTRYPOINT`指令后，用`CMD`指令指定具体参数；

`Dockerfile`中可以有多个`CMD`指令，但只有最后一个生效，`CMD`会被`docker run`后面跟的参数替换掉；

`RUN`指令是在`docker build`时执行，`CMD`是在`docker run`时执行；

```
CMD ./test.sh
CMD ["catalina.sh", "run"]
CMD ["run", "offline"] # 指定ENTRYPOINT的参数 
# 用下面的命令启动tomcat容器后，/bin/bash会替代tomcat Dockerfile中的CMD ["catalina.sh", "run"],
# 导致容器虽然启动成功了，但tomcat并没有启动
docker run -it -p 8080:8080 tomcat /bin/bash
```

### ENTRYPOINT

指定容器启动时要运行的命令，类似于`CMD`，但`ENTRYPOINT`不会被`docker run`后面的命令覆盖，而且`docker run`后面跟的命令行参数会被当做参数送给`ENTRYPOINT`指令指定的程序；

可以和`CMD`一起使用，一般是**变参**的时候才会用`CMD`给`ENTRYPOINT`传参，即当指定了`ENTRYPOINT`时，`CMD`的含义就发生了变化，不再是运行命令，而是将`CMD`的内容作为参数传递给`ENTRYPOINT`；

```
# 下面两个指令相当于执行命令nginx -c /etc/nginx/nginx.conf
ENTRYPOINT ["nginx", "-c"]
CMD ["/etc/nginx/nginx.conf"]
# 运行该命令时，CMD参数将被替换，实际执行的命令为 nginx -c /etc/nginx/new.conf
docker run -d nginx -c /etc/nginx/new.conf
```

## 自定义镜像

> 这里基于从docker hub上pull的centos镜像，使用Dockerfile自定义mycentos镜像，使之拥有jdk8环境

### 前提准备

拉好centos镜像，下载jdk8压缩包

### 步骤

```bash
# 1、编写Dockerfile
mkdir -p /tmp/myfile
cd /tmp/myfile
touch Dockerfile
vi Dockerfile
# 2、构建Dockerfile
# 在Dockerfile同级目录下执行命令
# 注意命令后有个空格和点号，表示当前目录
docker build -t mycentos:1.0 .
# 3、查看镜像并运行容器
docker images
docker run -it mycentos:1.0
# 4、测试jdk安装结果
java -version
```

### Dockerfile内容

```
# 基于centos镜像
FROM centos
# 作者和邮箱
MAINTAINER pxz<pxz@163.com>

# 设置MYPATH变量
ENV MYPATH /usr/local
# 设置默认工作目录
WORKDIR $MYPATH

# 创建java目录
RUN mkdir /usr/local/java
# 拷贝jdk压缩包到镜像，ADD命令会自动解压
# jdk的安装包需要和Dockerfile在同一目录
ADD OpenJDK8U-jdk_x64_linux_hotspot_8u322b06.tar.gz /usr/local/java
# 配置java环境变量
# jdk8u322-b06是jdk压缩包的解压目录
ENV JAVA_HOME /usr/local/java/jdk8u322-b06
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

# 暴露80端口
EXPOSE 80

# 容器启动后输出内容和执行命令
CMD echo $MYPATH
CMD echo "success............"
CMD /bin/bash
```

# 虚悬镜像

仓库名、标签都是`<none>`的镜像，俗称`dangling imag`，删除掉即可

```bash
# 查看所有虚悬镜像
docker image ls -f dangling=true
# 删除所有虚悬镜像
docker image prune
```

# docker network

## 作用

- 容器间的互联和通信以及端口映射
- 容器ip变动时，可直接通过服务名直接进行网络通信（需要自定义网络）

## 常用命令

`docker network --help`查看所有命令

```bash
# 查看docker network
docker network ls
# 查看网络源数据
docker network inspect networkid/networkname
# 创建网络
docker network create networkname
# 删除网络
docker rm networkid/networkname
```

## 网络模式

| 网络模式  |                             描述                             |                             命令                             |
| :-------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|  bridge   | 为每一个容器分配、设置ip，并将容器连接到一个`docker0`；<br />虚拟网桥，默认为该模式 |        使用`--network bridge`指定，默认使用`docker0`         |
|   host    | 容器不会虚拟出自己的网卡、配置自己的ip等，而是使用宿主机的ip和端口 |                   使用`--network host`指定                   |
|   none    | 容器有独立的`Network namespace`，但并没有对其进行任何的网络设置，如分配`veth pair`和网桥连接、ip等 |                   使用`--network none`指定                   |
| container | 新创建的容器不会创建自己的网卡和配置自己的ip，而是和一个指定的容器共享ip、端口范围等 |        使用`--network container:容器名称/容器id`指定         |
|  自定义   |                    自定义一个`network`，                     | `docker network create my_network`<br />`--network my_network` |

# docker compose

> Docker-Compose是Docker官网的开源项目，负责实现对Docker容器集群的快速编排。
>
> 官方文档：`https://docs.docker.com/compose/compose-file/compose-file-v3/`

## 安装docker compose

```bash
# 先检查下是否安装过，装docker时可能已经安装过了
yum list installed  | grep docker-compose
# 安装
yum -y install docker-compose-plugin
# 查看compose版本
docker compose version
```

## 核心概念

- 一个文件：`docker-compose.yml`
- 两个要素：
  - 服务service：一个个的容器实例，如订单服务、仓库服务、mysql容器等
  - 工程project：由一组关联的容器组成的一个**完整业务单元**，在`docker-compose.yml`文件中定义

## 使用步骤

1. 编写`Dockerfile`定义各个微服务应用，并构建出对应的镜像文件
2. 使用`docker-compose.yml`定义一个完整业务单元，安排好整体应用中的各个容器服务
3. 执行`docker compose up`命令，启动并运行整个应用程序，完成一键部署

## 常用命令

```bash
# 命令帮助
docker compose --help
# 创建并在后台启动容器
docker compose up -d
# 停止并删除容器、网络等
docker compose down
# 进入yml中配置的容器内部
docker compose exec 服务名 /bin/bash
# 查看yml中配置的所有容器
docker compose ps
# 查看yml中配置的所有容器进程
docker compose top
# 查看容器输出日志
docker compose logs 服务名
# 停止yml中配置的所有容器
docker compose stop
# 启动yml中配置的所有容器
docker compose start
# 重启yml中配置的所有容器
docker compose restart
# 检查yml配置，-q表示配置有误时输出
docker compose config -q
```

## 案例

> springboot+mysql+redis实现添加用户和查询用户的功能

### 准备工作

- 将编写好springboot项目打成jar包
- 参照[自定义镜像](#自定义镜像)构建好一个有jdk环境的centos镜像

### 步骤

#### 不使用docker compose

```bash
# 1、将springboot项目的jar包通过docker build构建成镜像
# 1.1、编写Dockerfile，文件内容见下文
mkdir /tmp/pxztest
vi Dockerfile
# 1.2、构建镜像
docker build -t userservice:1.0 .
# 2、启动mysql容器、redis容器和微服务容器
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=root -d -p 3306:3306 --privileged=true -v /tmp/mysql/log:/var/log/mysql -v /tmp/mysql/data:/var/lib/mysql -v /tmp/mysql/conf:/etc/mysql/conf.d mysql:5.7

docker run --name r1 -d -p 6379:6379 --privileged=true -v /tmp/redis/redis.conf:/etc/redis/redis.conf -v /tmp/redis/data:/data  redis:latest redis-server /etc/redis/redis.conf

docker run -d -p 8888:8888 --name user-service userservice:1.0
# 3、调用微服务接口并查看容器日志
curl -GET http://192.168.5.110:8888/find/4
docker logs -n50 -f user-service
```

#### 使用docker compose

> 不使用`docker compose`虽然可以成功运行案例，但是存在以下几个问题：
>
> 1. 容器启动的先后顺序要求固定，要先启动mysql+redis后，才能启动springboot；
> 2. 需要手动执行多个`docker run`命令；
> 3. 容器启停或宕机时，ip地址对应的容器实例可能发生变化，导致容器间互相访问失败，除非生产ip固定（不推荐），所以容器间需要通过服务名来访问，而不是ip；

**1、编写docker-compose.yml**

```bash
cd /tmp/pxztest
vi docker-compose.yml
```

***docker-compose.yml内容***

```yaml
version: "3" # 定义docker compose的版本

services: # 服务列表
  microService: # 自定义服务名称
    image: userservice:1.0 # 使用的镜像
    container_name: ms1 # 自定义容器名称
    ports: # 定义暴露的端口
      - "8888:8888"
    volumes: # 定义数据卷
      - /tmp/pxztest/userservice:/data
    networks: # 使用的docker network
      - pxz_network
    depends_on: # 依赖的服务
      - redis
      - mysql

  redis:
    image: redis:latest
    container_name: redis1
    ports:
      - "6379:6379"
    volumes:
      - /tmp/redis/redis.conf:/etc/redis/redis.conf
      - /tmp/redis/data:/data
    networks:
      - pxz_network
    command: redis-server /etc/redis/redis.conf

  mysql:
    image: mysql:5.7
    container_name: mysql1
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_ALLOW_ENPTY_PASSWORD: "no"
      MYSQL_DATABASE: "db1"
    ports:
      - "3306:3306"
    volumes:
      - /tmp/mysql/log:/var/log/mysql
      - /tmp/mysql/data:/var/lib/mysql
      - /tmp/mysql/conf:/etc/mysql/conf.d
      - /tmp/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - pxz_network
    command: --default-authentication-plugin=mysql_native_password # 解决外部无法访问

networks: # 创建network
  pxz_network:
```

**2、调整springboot项目**

将springboot项目中mysql和redis的ip地址替换为`docker-compose.yml`中定义的服务名，并重新打包、生成镜像。

**3、启动docker compose**

```bash
# 在docker-compose.yml同级目录下执行
docker compose up -d
# 查看结果
docker compose ps
```

#### Dockerfile内容

```
# mycentos镜像要有jdk环境
FROM mycentos:1.0
ADD dockerboot-0.0.1-SNAPSHOT.jar /tmp/service/pxz.jar
EXPOSE 8888
CMD java -jar /tmp/service/pxz.jar
```

# 可视化工具Portainer

> 官网：`https://www.portainer.io/`
>
> 官方文档：`https://docs.portainer.io/v/ce-2.11/start/install`

## 使用

```bash
# 1、启动portainer --restart表示该容器会跟随docker服务的重启而重启，永远在running状态
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.11.1
# 2、访问web页面
https://192.168.5.110:9443/
# 3、第一次登录需要创建用户
admin/12345678

```

# docker容器监控CIG

> CIG = CAdvisor（监控收集） + InfluxDB（存储数据） + Granfana（展示图表）

## 配合docker compose使用CIG

```bash
mkdir /tmp/cig
cd /tmp/cig
vi docker-compose.yml # yml内容见下文
# 检查yml文件配置，无输出表示配置正确
docker compose config -q
# 启动compose，在yml文件同级目录下执行
# 启动granfana报错：GF_PATHS_DATA='/var/lib/grafana' is not writable.
# 问题原因：yml中配置的104用户没有权限
# 解决方法：给yml中与GF_PATHS_DATA映射的数据卷目录加777权限即可
docker compose up -d
docker compose ps
# 浏览器访问
http://192.168.5.110:8080/ # 访问CAdvisor
http://192.168.5.110:8083/ # 访问influxdb
http://192.168.5.110:3000/ # 访问granfana，admin/admin
```

### 配置granfana

#### 配置数据源

![配置数据源1](1653811219(1).jpg)

![配置数据源2](1653811302(1).jpg)

#### 添加panel

![添加panel1](1653811455(1).jpg)

![添加panel2](1653811630(1).jpg)

![添加panel3](1653814050(1).jpg)

### docker-compose.yml内容

```yaml
version: "3"

volumes:
  grafana_data: {}

services:
  influxdb:
    image: tutum/influxdb:0.9
    restart: always
    environment:
      - PRE_CREATE_DB=cadvisor
    ports:
      - "8083:8083"
      - "8086:8086"
    volumes:
      - /tmp/influxdb:/data
  cadvisor:
    image: google/cadvisor
    links:
      - influxdb:influxsrv
    command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  grafana:
    user: "104"
    image: grafana/grafana
    restart: always
    links:
      - influxdb:influxsrv
    ports:
      - "3000:3000"
    volumes:
      - /var/lib/grafana:/var/lib/grafana
    environment:
      - HTTP_USER=admin
      - HTTP_PASS=admin
      - INFLUXDB_HOST=influxsrv
      - INFLUXDB_PORT=8086
```

# 卸载docker

```bash
systemctl stop docker
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

