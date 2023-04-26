---
title: Nacos入门
date: 2022-08-16 15:40:28
categories:
  - 微服务
tags:
  - Nacos
---

[TOC]

# 简介

> 阿里开源的组件，可与**Spring**、**SpringBoot**、**SpringCloud**、**Docker**、**K8s**等集成，实现**服务注册与发现**、**配置管理**等功能。
>
> 官网地址：`https://nacos.io/zh-cn/index.html`
>
> github：`https://github.com/alibaba/nacos`

# 集群环境搭建

## 环境准备

1. 64 bit OS Linux/Unix/Mac，推荐使用Linux系统。
2. 64 bit JDK 1.8+。
3. 3个或3个以上Nacos节点才能构成集群。

### linux配置jdk环境

**下载jdk**

```http
# openjdk下载地址
https://adoptium.net/zh-CN/
```

**解压jdk**

```bash
tar -zxvf OpenJDK8U-jdk_x64_linux_hotspot_8u322b06.tar.gz
```

**设置环境变量**

```bash
vi ~/.bash_profile
```

```
# 在文件后添加如下设置，注意路径
JAVA_HOME=/home/nacos/jdk8u322-b06
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME
export PATH
```

```bash
# 使环境变量生效
source ~/.bash_profile
# 查看环境变量
echo $PATH
```

## 集群架构

![nacos集群架构](nacos集群架构.png)

## 搭建过程

**下载安装包**

地址：`https://github.com/alibaba/nacos/releases`

本次下载的是2.1.0版本

---

**解压安装包**

```bash
tar -zxvf nacos-server-2.1.0.tar.gz
```

---

**配置集群配置文件**

三个节点各自修改配置文件`nacos/conf/cluster.conf`：

```
# ip:port
172.19.4.41:8848
172.19.4.42:8848
172.19.4.43:8848
```

---

**设置数据源为mysql**

> nacos默认使用内置数据源，集群环境下各节点数据可能不一致，所以需要切换为外置数据源，目前只支持mysql。

执行sql脚本`conf/nacos-mysql.sql`；

```sql
# 先创建数据库
CREATE DATABASE nacos CHARSET utf8;
USE nacos;
# 执行脚本，注意脚本路径
source nacos-mysql.sql
```

三个节点分别修改配置文件`conf/application.properties`;

```properties
# 这几项在配置文件中都被注释掉了，这里需要去掉注释，并修改属性值即可
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://172.19.4.37:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=itc123
```

---

**配置nginx**

修改`conf/nginx.conf`配置文件

```
# 需要有nginx stream模块，用于转发nacos2.0版本的grpc请求
stream {
	
	upstream nacos-server-grpc {
		server 172.19.4.41:9848;
		server 172.19.4.42:9848;
		server 172.19.4.43:9848;
	}
	
	server {
		listen			9848;
		proxy_pass http://nacos9848;
	}

}

http {
	
	# 其他内容

    upstream nacos {
        server 172.19.4.41:8848;
        server 172.19.4.42:8848;
        server 172.19.4.43:8848;
    }

    server {
        listen			8848;
        server_name		nacos_server;
        location / {
            proxy_pass http://nacos;
        }
    }
    
    # 其他内容
}

```

启动nginx

`nginx/sbin/nginx`

---

**启动nacos服务器**

三个节点分别执行：

```bash
cd nacos
sh bin/startup.sh
```

---

**验证集群**

控制台访问

```
url:http://nginxip:8848/nacos
account:nacos/nacos
```

![nacos控制台访问](nacos控制台访问.png)

![nacos集群节点](nacos集群节点.png)

# 使用Nacos作为配置中心

## 说明

- nacos用**namespace + group id + data id**来标识唯一的配置；

![标识唯一配置](标识唯一配置.png)

- namespace可用于区分环境，如dev、test、pro等；
- 支持text、json、xml、yaml、html、properties等格式的配置文件；

## 与Spring集成

> 官方地址：`https://github.com/nacos-group/nacos-spring-project`

### 版本

```
jdk: 1.8+
spring: 3.2+
```

**Maven坐标**

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba.nacos/nacos-spring-context -->
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-spring-context</artifactId>
    <version>1.1.1</version>
</dependency>
```

---

**配置类**

```java
@Configuration
// 开启nacos配置
@EnableNacosConfig(globalProperties = @NacosProperties(
        serverAddr = "172.19.4.41:8848,172.19.4.42:8848,172.19.4.43:8848",
        username = "nacos", password = "nacos", namespace = "9e50602c-2575-4761-8166-290af5da5044"
))
// 通过groupId和dataId加载配置文件
@NacosPropertySource(groupId = "DEFAULT_GROUP", dataId = "ir1", type = ConfigType.YAML, autoRefreshed = true)
@NacosPropertySource(groupId = "DEFAULT_GROUP", dataId = "ir2", type = ConfigType.YAML, autoRefreshed = true)
public class NacosConfig {
}
```

---

**nacos配置**

在`dev`下添加配置：

![添加配置](添加配置.png)



`ir1`配置详情：

![配置详情1](配置详情1.png)



`ir2`配置详情：

![配置详情2](配置详情2.png)





---

**获取配置**

```java
@Controller
@RequestMapping("config")
public class ConfigController {
	
    // 使用@NacosValue注解获取配置，开启自动刷新
    @NacosValue(value = "${sso.service}", autoRefreshed = true)
    private String ssoService;
    @NacosValue(value = "${sso.server.login-url}", autoRefreshed = true)
    private String ssoServerLoginUrl;
    @NacosValue(value = "${sso.server.prefix-url}", autoRefreshed = true)
    private String ssoServerPrefixUrl;
    // 默认值为hi
    @NacosValue(value = "${hello:hi}", autoRefreshed = true)
    private String hello;

    @RequestMapping(value = "/get", method = GET)
    @ResponseBody
    public boolean get() {
        return ssoService + " - " + ssoServerLoginUrl + " - " + 
            ssoServerPrefixUrl + " - " + hello;
    }
}
```

浏览器访问`http://ip:port/config/get`；

---

**修改配置**

在控制台修改配置：

![编辑配置](编辑配置.png)

---

**再次获取配置**

浏览器再次访问`http://ip:port/config/get`，此时获取到更新后的配置。

## 与Spring Boot集成

>官方地址：`https://github.com/nacos-group/nacos-spring-boot-project`

**Maven坐标**

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba.boot/nacos-config-spring-boot-starter -->
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.11</version>
</dependency>
```

---

**application.yml配置**

```yaml
nacos:
  config:
    # nacos server地址
    # 因为nginx没有配置stream模块，所以没有使用nginx代理的地址，而是直接使用nacos集群的地址
    server-addr: 172.19.4.41:8848,172.19.4.42:8848,172.19.4.43:8848
    bootstrap:
      enable: true # 开启配置预加载功能
    remote-first: true # 设置nacos上的配置优先于本地配置
    namespace: 9e50602c-2575-4761-8166-290af5da5044 # 命名空间id
    group: group1 # group id
    data-ids: test1.yml # data id，多个用逗号隔开
    type: yaml # 配置文件类型
    auto-refresh: true  # 开启自动刷新
    max-retry: 10 # 最大重试次数
    config-retry-time: 2333 # 重试时间
    config-long-poll-timeout: 46000 # 监听长轮询超时时间
    username: nacos # 用户名
    password: nacos # 密码
    # 加载其他配置，类似@NacosPropertySource()
    ext-config:
      - namespace: 9e50602c-2575-4761-8166-290af5da5044
        group: group2
        data-ids: test2.yml
        type: yaml
        auto-refresh: true
      - namespace: 9e50602c-2575-4761-8166-290af5da5044
        group: group3
        data-ids: test3.yml
        type: yaml
        auto-refresh: true
```

---

**nacos配置**

![配置示例1](配置示例1.png)

![配置示例2](配置示例2.png)



![配置示例3](配置示例3.png)



![配置示例4](配置示例4.png)

**获取属性值**

```java
import com.alibaba.nacos.api.config.ConfigType;
import com.alibaba.nacos.api.config.annotation.NacosConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@NacosConfigurationProperties(groupId = "group1", dataId = "test1.yml", prefix = "people", type = ConfigType.YAML, autoRefreshed = true)
public class PeopleConfig {

    private String name;
    private int age;
    private String sex;

    // getter、setter、toString
}
```

```java
import com.alibaba.nacos.api.config.annotation.NacosValue;
import com.lhx.nacosconfigdemo.config.PeopleConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/config")
public class ConfigController {

    @NacosValue(value = "${hello:hi}", autoRefreshed = true)
    private String hello;
    @NacosValue(value = "${count:-1}", autoRefreshed = true)
    private int count;
    @NacosValue(value = "${ping:unknow}", autoRefreshed = true)
    private String ping;
    @Autowired
    private PeopleConfig peopleConfig;

    @GetMapping("/get")
    public String get() {
        System.out.println(peopleConfig);
        System.out.println(hello);
        System.out.println(count);
        System.out.println(ping);
        return "success";
    }

}
```

浏览器请求`http://localhost:8080/config/get`;

---

**编辑配置**

 在控制台编辑配置：

![编辑配置2](编辑配置2.png)

---

**再次获取配置**

浏览器再次请求`http://localhost:8080/config/get`，成功从配置中心获取到更新后的配置；

# 维护Nacos配置中心

> 方法：
>
> - 登录到nacos控制台维护配置；
> - 使用open api维护配置；

## 使用控制台编辑配置

> 控制台手册：`https://nacos.io/zh-cn/docs/console-guide.html`

浏览器访问`http://nacos-server-ip:port/nacos`，输入初始账号密码`nacos/nacos`后登录到nacos控制台；

![控制台登录](控制台登录.png)

在控制台**配置管理**处编辑配置信息；

![编辑配置3](编辑配置3.png)

## 使用Open API编辑配置

> api指南：`https://nacos.io/zh-cn/docs/open-api.html`

### 获取配置

请求方式：GET

URL：/nacos/v1/cs/configs

请求参数：

| 名称   | 类型   | 是否必须 | 描述                                    |
| :----- | :----- | :------- | :-------------------------------------- |
| tenant | string | 否       | 租户信息，对应 Nacos 的命名空间ID字段。 |
| dataId | string | 是       | 配置 ID。                               |
| group  | string | 是       | 配置分组。                              |

```bash
curl -X GET 'http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.example&group=com.alibaba.nacos'
```

### 发布/更新配置

请求方式：POST

URL：/nacos/v1/cs/configs

请求参数：

| 名称    | 类型   | 是否必须 | 描述                                          |
| :------ | :----- | :------- | :-------------------------------------------- |
| tenant  | string | 否       | 租户信息，对应 Nacos 的命名空间ID字段         |
| dataId  | string | 是       | 配置 ID                                       |
| group   | string | 是       | 配置分组                                      |
| content | string | 是       | 配置内容                                      |
| type    | String | 否       | 配置类型，yaml、json、xml、text、properties等 |

```bash
curl -X POST 'http://127.0.0.1:8848/nacos/v1/cs/configs' -d 'dataId=nacos.example&group=com.alibaba.nacos&content=contentTest'
```

### 删除配置

请求方式：DELETE

URL：/nacos/v1/cs/configs

请求参数：

| 名称   | 类型   | 是否必须 | 描述                                  |
| :----- | :----- | :------- | :------------------------------------ |
| tenant | string | 否       | 租户信息，对应 Naocs 的命名空间ID字段 |
| dataId | string | 是       | 配置 ID                               |
| group  | string | 是       | 配置分组                              |

```bash
curl -X DELETE 'http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.example&group=com.alibaba.nacos'
```

