---
title: RabbitMQ入门
date: 2022-08-16 14:52:23
categories:
  - 中间件
  - 消息队列
tags:
  - RabbitMQ
  -	MQ
---

# 1. 搭建RabbitMQ单机环境

## 1.1. 版本

   	Linux：CentOS7
   	Erlang：23.3.4
   	RabbitMQ：3.9.13

## 1.2. 资源下载

	Erlang rpm包下载地址：https://github.com/rabbitmq/erlang-rpm/releases
	RabbitMQ rpm包下载地址：https://github.com/rabbitmq/rabbitmq-server/releases

## 1.3. 搭建步骤

### 1.3.1. 安装Erlang

**_RabbitMQ依赖Erlang环境，安装RabbitMQ之前，先要安装Erlang环境。_**

```
# 将下载好的Erlang rpm包放到/usr/local/erlang目录下，执行
rpm -ivh erlang-23.3.4.4-1.el7.x86_64.rpm
# 查看Erlang版本
erl -version
```

![安装Erlang](安装Erlang.png)

### 1.3.2. 安装socat

**_安装RabbitMQ之前还需要安装一下socat。_**

```
# 安装socat
yum install socat -y
```

![安装socat](安装socat.png)
_执行`yum install socat -y`时，若出现`Cannot find a valid baseurl for repo: base/7/x86_64`错误，可尝试通过配置DNS来解决，具体如下：_

```
# 编辑ifcfg-ens33文件（不同系统该文件名称可能不同）
vi /etc/sysconfig/network-scripts/ifcfg-ens33
# 在ifcfg-ens33中追加配置
DNS1=8.8.8.8 
DNS2=4.2.2.2
# 重启网络
systemctl restart network.service
```

![配置DNS](配置DNS.png)

### 1.3.3. 安装RabbitMQ

```
# 将下载好的RabbitMQ rpm包放到/usr/local/rabbitmq目录下，执行
rpm -ivh rabbitmq-server-3.9.13-1.el7.noarch.rpm
# 添加开机启动
chkconfig rabbitmq-server on
```

![安装RabbitMQ](安装RabbitMQ.png)

```
# 添加web管理页面插件
rabbitmq-plugins enable rabbitmq_management
```

![添加web管理页面插件](添加web管理页面插件.png)


```
# 启动RabbitMQ服务，虽然已经设置了开机启动，但本次还得手动起一下
# 查看服务状态
/sbin/service rabbitmq-server status
# 启动服务
/sbin/service rabbitmq-server start
# 停止服务
/sbin/service rabbitmq-server stop
```

![启停服务](启停服务.png)
	
**_添加登录用户，内置的用户guest/guest只能在本机登录，要访问Linux下的服务，需要额外添加用户。_**

```
# 用户名密码分别为admin/admin，拥有所有权限
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 查看用户列表
rabbitmqctl list_users
```

![添加用户](添加用户.png)

### 1.3.4. 关闭防火墙

**_在windows下访问Linux上的服务，需要先关闭Linux上的防火墙。_**

```
# 查看防火墙状态
systemctl status firewalld
# 临时关闭防火墙（系统重启后仍会开启防火墙）
systemctl stop firewalld
# 永久关闭防火墙
systemctl disable firewalld
```

![关闭防火墙](关闭防火墙.png)

### 1.3.5. 测试安装结果

**_浏览器访问`http://ip地址:15672/`，输入用户名密码`admin/admin`,能访问到RabbitMQ主页即为安装成功。_**
![登录RabbitMQ](登录RabbitMQ.png)
![RabbitMQ主页](RabbitMQ主页.png)

# 2. 搭建RabbitMQ集群环境

## 2.1. 搭建环境

   	三台搭建好RabbitMQ单机环境的服务器，单机环境搭建步骤见上文。

## 2.2. 搭建步骤

```
# 修改三台服务器的主机名，分别为node1、node2、node3
vi /etc/hostname
# 修改完主机名后重启三台服务器使之生效
reboot
```

![修改主机名](修改主机名.png)

```
# 修改各节点的hosts文件，使各节点能互相访问
vi /etc/hosts
# 添加配置
ip地址1 node1
IP地址2 node2
ip地址3 node3
```

![修改hosts文件](修改hosts文件.png)

```
# 三个节点使用相同的cookie，在node1上分别执行（该命令要输入node2、node3的用户密码）：
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie

```

![统一各节点的cookie](统一各节点的cookie.png)

```
# 将node2加入到node1，node3加入到node2，以构成三个节点的集群，操作如下：

# 在node2上执行(stop会关闭erlang虚拟机，stop_app只关闭rabbitmq服务)
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app
# 在node3上执行
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node2
rabbitmqctl start_app

# 至此，三个节点的集群环境已搭建完毕
# 查看集群状态
rabbitmqctl cluster_status
```

![查看集群状态](查看集群状态.png)

```
# 剔除集群节点（下线的节点可以剔除掉）
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl forget_cluster_node rabbit@node1
rabbitmqctl start_app
```

**_单机环境和集群环境下用户不通用，所以要添加集群用户，方式同“添加单机用户”_**

## 2.3. 验证集群效果

**_随便访问一台单机的RabbitMQ页面，输入集群用户名`admin/admin`后，看到三个健康的节点即为成功。_**
![RabbitMQ集群验证](RabbitMQ集群验证.png)

## 2.4. 添加镜像队列策略

**各参数说明**

- *Name：策略名称，可随意指定，不重复即可。*
- Pattern：匹配策略，为正则表达式，如`^mirror`表示所有名称以`mirror`开头的队列均为镜像队列。
- Apply to：应用于何处。
- Definition:
  - ha-mode：exactly，表示模式为指定模式。
  - ha-params：2，表示备份数量为2。
  - ha-sync-mode：automatic，表示同步策略为自动同步。
    ![添加镜像队列](添加镜像队列.png)

# 3. 卸载RabbitMQ和Erlang

## 3.1. 卸载RabbitMQ

```
# 停止RabbitMQ服务
/sbin/service rabbitmq-server stop
# 查看rabbitmq安装的相关列表
yum list | grep rabbitmq
# 卸载rabbitmq已安装的相关内容
yum -y remove rabbitmq-server.noarch
# 删除相关文件
rm -rf /var/lib/rabbitmq
```

## 3.2. 卸载Erlang

```
# 查看erlang安装的相关列表
yum list | grep erlang
# 卸载erlang已安装的相关内容
yum -y remove erlang-*
yum remove erlang.x86_64
# 删除相关文件
rm -rf /usr/lib64/erlang
```

