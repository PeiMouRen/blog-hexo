---

title: FastDFS入门
date: 2022-08-10 18:39:41
categories: 
  - 中间件
  -	分布式文件存储
tags:
 - FastDFS
---

[TOC]

# 1. fastdfs简介

> ​		FastDFS是一款开源的分布式文件系统，功能主要包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了文件大容量存储和高性能访问的问题。FastDFS特别适合以文件为载体的在线服务，如图片、视频、文档等等。
>
> ​		FastDFS作为一款轻量级分布式文件系统，版本V6.01代码量6.3万行。FastDFS用C语言实现，支持Linux、FreeBSD、MacOS等类UNIX系统。FastDFS类似google FS，属于应用级文件系统，不是通用的文件系统，只能通过专有API访问，目前提供了C和Java SDK，以及PHP扩展SDK。
>
> ​		FastDFS为互联网应用量身定做，解决大容量文件存储问题，追求高性能和高扩展性。FastDFS可以看做是基于文件的key value存储系统，key为文件ID，value为文件内容，因此称作分布式文件存储服务更为合适。

## 1.1. 特点

- 分组存储，简单灵活；
- 对等结构，不存在单点；
- 文件ID由FastDFS生成，作为文件访问凭证。FastDFS不需要传统的name server或meta server；
- 大、中、小文件均可以很好支持，可以存储海量小文件；
- 一台storage支持多块磁盘，支持单盘数据恢复；
- 提供了nginx扩展模块，可以和nginx无缝衔接；
- 支持多线程方式上传和下载文件，支持断点续传；
- 存储服务器上可以保存文件附加属性;

## 1.2. 设计理念

​		FastDFS 系统有三个角色：跟踪服务器(Tracker Server)、存储服务器(Storage Server)和客户端(Client)。

　　**Tracker Server**：跟踪服务器，主要做调度工作，起到均衡的作用；负责管理所有的 storage server和 group，每个 storage 在启动后会连接 Tracker，告知自己所属 group 等信息，并保持周期性心跳。通过Tracker server在文件上传时可以根据一些策略找到Storage server提供文件访问服务。

　　**Storage Server**：存储服务器，主要提供容量和备份服务；以 group 为单位，每个 group 内可以有多台 storage server，数据互为备份。Storage server没有实现自己的文件系统而是利用操作系统 的文件系统来管理文件。

　　**Client**：客户端，上传下载数据的服务器，也就是我们自己的项目所部署在的服务器。

## 1.3. 文件交互过程

### 1.3.1 上传文件

1. tracker server收到上传请求后，查询可用的storage server，并返回给client；

2. client直接跟storage server建立连接，进行文件上传；

3. storage server生成文件id返回给client；

![fastdfs上传文件过程](fastdfs上传文件过程.png)

### 1.3.2 下载文件

1. tracker server收到下载请求后，查询可用的storage server，并返回给client；
2. client与storage server建立连接，并附带文件id；
3. storage server根据文件id查找文件，并将文件返回给client；

![fastdfs下载文件过程](fastdfs下载文件过程.jpg)

## 1.4. 架构图

![fastdfs架构图](fastdfs架构图.png)

## 1.5. 名词解释

**tracker server**：中心服务器，FastDFS server端两大主角之一。tracker server作为FastDFS集群管理中心，管								理集群拓扑结构，对storage server文件上传和文件下载起到负载均衡调度作用。使用命令								fdfs_monitor可以查看集群状态。

**storage server**：存储服务器，FastDFS server端两大主角之一。文件相关功能都通过storage server来完成，包								括文件上传、下载、文件同步等等。

**文件ID**：上传文件时由storage server生成访问该文件的凭证，包括group名称、存储路径顺序号以及包含两级目				录的文件名，应用程序（调用方）需要将文件ID保存到数据库等存储介质中。

​				![fastdfs文件id详解](fastdfs文件id详解.png)

**group**：FastDFS采用了分组存储方式，storage集群由一个或多个组构成，storage集群存储总容量为storage集				群中所有组的存储容量之和；一个组由一台或多台storage存储服务器组成，同组内的多台Storage 				server之间是类似RAID1的互备关系，同组存储服务器上的文件是完全一致的。文件上传、下载、删除等				操作可以在组内任意一台Storage server上进行。类似木桶短板效应，一个组的存储容量为该组内存储服				务器容量最小的那个。

​	

# 2. fastdfs单机环境搭建

## 2.1. 说明

若安装过程中提示没有权限，可切换到root用户进行安装。

单机环境下，服务器同时担任tracker角色和storage角色，需要安装的应用及其对应的说明如下：

- **libfastcommon**：fastdfs依赖的函数库；

  > c common functions library extracted from my open source project FastDFS. this library is very simple and stable. functions including: string, logger, chain, hash, socket, ini file reader, base64 encode / decode, url encode / decode, fast timer, skiplist, object pool etc. detail info please see the c header files.
  >
  > 

- **libserverframe**：从FastDFS继承过来的网络服务框架库，FastDFS6.0.9版本及后续版本后需要安装这个。

- **fastdfs**：分布式文件系统；

- **fastdfs-nginx-module**：nginx的一个扩展模块，用来解决**storage cluster同组内文件异步复制带来延											迟导致文件访问不到**问题，该模块需要与nginx配合使用；

- **nginx**：负载均衡服务器，在storage server上与fastdfs-nginx-module配合使用；在tracker server上只是将请求转发到storage server；

**<font color="red">注意：fastdfs-nginx-module主要是解决storage cluster集群的问题，若集群环境下，tracker cluster中的服务器无需安装。</font>**


## 2.2. 版本

```
linux:centos7
libfastcommon:1.0.62
libserverframe：1.1.21
fastdfs:6.09
fastdfs-nginx-module:1.23
nginx:1.22.0
```

## 2.3. 资源下载

``` http
# libfastcommon下载地址
https://github.com/happyfish100/libfastcommon/tags
# libserverframe下载地址
https://github.com/happyfish100/libserverframe/tags
# fastdfs下载地址
https://github.com/happyfish100/fastdfs/tags
# fastdfs-nginx-module下载地址
https://github.com/happyfish100/fastdfs-nginx-module/tags
# nginx下载地址
http://nginx.org/
```

## 2.4. 编译环境

```bash
yum install gcc gcc-c++ kernel-devel make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
```

## 2.5. 安装

```bash
# 安装libfastcommon
tar -zxvf libfastcommon-1.0.62.tar.gz
cd libfastcommon-1.0.62
./make.sh && ./make.sh install

# 安装libserverframe
tar -zxvf libserverframe-1.1.21.tar.gz
cd libserverframe-1.1.21
./make.sh && ./make.sh install

# 安装fastdfs
tar -zxvf fastdfs-6.09.tar.gz
cd fastdfs-6.09
./make.sh && ./make.sh install

# 安装fastdfs-nginx-module（解压即可，后续安装nginx时会用到）
tar -zxvf fastdfs-nginx-module-1.23.tar.gz

# 安装nginx
tar -zxvf nginx-1.22.0.tar.gz
cd nginx-1.22.0
# 添加配置（集群环境下在tracker server上安装时无需加--add-module参数）
# --add-module后面的路径换成自己安装fastdfs-nginx-module下的src目录
# nginx会默认安装到/usr/local/nginx目录
./configure --add-module=/usr/local/fastdfs/fastdfs-nginx-module-1.23/src/
make && make install
```

## 2.6. 编辑配置文件

### 2.6.1. 说明

上述资源安装完毕后，会在`/etc/fdfs/`目录下生成相应的配置文件，各文件含义如下：

- **client.conf**：客户端配置文件，在服务器上使用该文件可直接上传资源，无需借助客户端（java、C等）工具。

- **tracker.conf**：tracker server相关的配置文件；
- **storage.conf**：storage server相关的配置文件；
- **storage_ids.conf**：storage server的id配置文件，可将storage server的group name和ip映射为id；该文件默认不生效，需在tracker.conf中指定`use_storage_id`为`true`，且`storage_ids_filename`为该文件的name才可使用；

除了上述配置文件外，`storage server`还需要在`/etc/fdfs/`目录下手动添加以下几个配置文件：

- **http.conf**：http配置文件，从`fastdfs`安装目录下的`conf/`中复制过来即可，后续开启[防盗链](#6.2. 防盗链)时，需要配置；

- **mime.types**：http配置文件，从`fastdfs`安装目录下的`conf/`中复制过来即可，后续无需配置；

- **mod_fastdfs.conf**：`fastdfs-nginx-module`配置文件，从`fastdfs-nginx-module`安装目录下的`src/`中复制过来即可；

  ``` bash
  # 添加storage server需要使用的配置文件
  # 注意文件路径替换成自己的
  cp /usr/local/fastdfs/fastdfs-6.09/conf/http.conf /etc/fdfs/
  cp /usr/local/fastdfs/fastdfs-6.09/conf/mime.types /etc/fdfs/
  cp /usr/local/fastdfs/fastdfs-nginx-module-1.23/src/mod_fastdfs.conf /etc/fdfs/
  ```

### 2.6.2. 配置client.conf

**<font color="red">非必须，直接在服务器上使用命令上传文件时再配置即可。</font>**

``` bash
# 主要配置以下几项

# 存储日志的目录，默认为/home/yuqing/fastdfs,该目录需要手动创建
base_path = /home/yuqing/fastdfs
# tracker server的地址，tracker cluster时可指定多个
tracker_server = 192.168.5.112:22122
```

### 2.6.3. 配置tracker.conf

```bash
# 主要配置以下几项

# 是否禁用配置文件，默认为false
disabled = false
# tracker server端口号，默认为22122
port = 22122
# 存储数据和日志的目录，默认为/home/yuqing/fastdfs,该目录需要手动创建
base_path = /home/yuqing/fastdfs
# http服务端口号，默认为8080
http.server_port = 8080
```

### 2.6.4. 配置storage.conf

```http
# storage详细配置列表（配置文件中也有详细的注释）
https://github.com/happyfish100/fastdfs/wiki/Configuration#storage
```

```bash
# 主要配置以下几项

# 是否禁用配置文件，默认为false
disabled = false
# 本机storage server所在的组
group_name = group1
# storage server端口号，默认为23000
port = 23000
# 存储数据和日志的目录，默认为/home/yuqing/fastdfs,该目录需要手动创建
base_path = /home/yuqing/fastdfs
# 存储路径个数，默认为1
store_path_count = 1
# 存储路径，根据store_path_count的个数可指定多个，如store_path1、store_path2...
store_path0 = /home/yuqing/fastdfs
# tracker server的地址，tracker cluster时可指定多个
tracker_server = 192.168.5.112:22122
# http服务端口号，默认为8888
http.server_port = 8888
```

### 2.6.5. 配置mod_fastdfs.conf

```bash
# 主要配置以下几项

# 存储日志文件的目录，默认为/tmp
base_path=/tmp
# tracker server的地址，tracker cluster时可指定多个
tracker_server=192.168.5.112:22122
# storage server端口号，默认为23000
storage_server_port=23000
# 本机storage server所在的组
group_name=group1
# 返回的文件id中是否含有group name，默认为false，这里改成true	
url_have_group_name = true
# 存储路径个数，默认为1，需要与本机storage.conf中的配置一致
store_path_count=1
# 存储路径，根据store_path_count的个数可指定多个，如store_path1、store_path2...
# 需要与本机storage.conf中的配置一致
store_path0=/home/yuqing/fastdfs
```

### 2.6.6. 配置nginx.conf

**注意：由于`storage server`上配置了`fastdfs-nginx-module`，所以需要将访问资源的请求`/group[0-9]`转发到`ngx_fastdfs_module`处理；而`tracker server`只需将访问资源的请求转发到`storage server`即可；**

```bash
# 编辑nginx.conf
vi /usr/local/nginx/conf/nginx.conf

# storage server上的nginx配置
# 主要调整listen和location
# 注意，这里的8888端口号要与/etc/fdfs/storage.conf中http.server_port的配置保持一致
http {
    server {
        listen       8888;    ## 该端口为storage.conf中的http.server_port相同
        server_name  localhost;
        location ~/group[0-9]/ {
            ngx_fastdfs_module;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
        root   html;
        }
    }
}

# tracker server上的nginx配置
http {

	upstream fastdfs_group_server {
        server 192.168.5.110:8888;
        server 192.168.5.111:8888;
	}
    server {
        listen       80;    
        server_name  localhost;
        location ~/group[0-9]/ {
            proxy_pass http://fastdfs_group_server;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
        root   html;
        }
    }
}
```

## 2.7. 测试搭建结果

```bash
# 启动tracker、storage和nginx
fdfs_trackerd /etc/fdfs/tracker.conf start
fdfs_storaged /etc/fdfs/storage.conf start
/usr/local/nginx/sbin/nginx
# 查看fdfs集群状态
fdfs_monitor /etc/fdfs/storage.conf
# 上传文件
fdfs_upload_file /etc/fdfs/storage.conf /usr/local/1.jpg
# 返回如下文件id
group1/M00/00/00/wKgFbmJeWImAVvMQAADX1NYCr6c251.jpg
# 关闭防火墙（或者开放端口tracker server和storage server占用的端口）
systemctl stop firewalld
# 通过浏览器获取上传的资源
http://192.168.5.112:8888/group1/M00/00/00/wKgFbmJeWImAVvMQAADX1NYCr6c251.jpg
```

# 3. fastdfs集群环境搭建

## 3.1. 说明

​		使用6台机器搭建集群，6台机器分别为node1、node2、node3、node4、node5、node6，其中node1和node2为tracker server，其他4台机器都为storage server，node3和node4为group1，node5和node6为group2。

​		6台机器分别安装好**单机部署**所述资源（`tracker server`不用安装`fastdfs-nginx-module`）；

![fastdfs搭建集群架构](fastdfs集群架构v2.png)

## 3.2. tracker.conf配置

两台`tracker server`上的`/etc/fdfs/tracker.conf`配置同**单机部署**，示例见下：

```bash
disabled = false
port = 22122
base_path = /home/yuqing/fastdfs
http.server_port = 8080
```

## 3.3. storage.conf配置

四台`storage server`上的`/etc/fdfs/storage.conf`与单机有两处不同：

- `group_name`需要特殊指定，即两个节点指定为`group1`，另外两个节点指定为`grorup2`;
- `tracker_server`需要指定多个（如果存在多个的话）；

```bash
disabled = false
# node3和node4指定为group1，node5和node6指定为group2
group_name = group1
port = 23000
base_path = /home/yuqing/fastdfs
store_path_count = 1
store_path0 = /home/yuqing/fastdfs
# 存在多个tracker server时需全部配置
tracker_server = node1:22122
tracker_server = node2:22122
http.server_port = 8888
```

## 3.4. mod_fastdfs.conf配置

四台`storage server`上的`/etc/fdfs/mod_fastdfs.conf`与`/etc/fdfs/storage.conf`类似，同样是`group_name`和`tracker_server`的配置与**单机**稍有不同：

```bash
base_path=/tmp
# 存在多个tracker server时需全部配置
tracker_server = node1:22122
tracker_server = node2:22122
storage_server_port=23000
# node3和node4指定为group1，node5和node6指定为group2
group_name=group1
url_have_group_name = true
store_path_count=1
store_path0=/home/yuqing/fastdfs
```

## 3.5. 测试搭建结果

```bash
# 启动两台tracker server
fdfs_trackerd /etc/fdfs/tracker.conf start
# 启动四台storage server
fdfs_storaged /etc/fdfs/storage.conf start
# 启动nginx
/usr/local/nginx/sbin/nginx -s reload
# 查看集群状态
fdfs_monitor /etc/fdfs/storage.conf
# 在node1上上传11.jpg
fdfs_upload_file /etc/fdfs/client.conf /usr/local/11.jpg
# 返回11.jpg文件id
group2/M00/00/00/wKgFbmJfdTyAUnbkAAFLdk23Zbg817.jpg
# 在node2上上传22.jpg
fdfs_upload_file /etc/fdfs/client.conf /usr/local/22.jpg
# 返回22.jpg文件id
group1/M00/00/00/wKgFb2JfdVGASCFEAAAz0tyYoGo988.jpg
# 浏览器访问上传的资源
http://192.168.5.11:8888/group1/M00/00/00/wKgFb2JfdVGASCFEAAAz0tyYoGo988.jpg
http://192.168.5.11:8888/group2/M00/00/00/wKgFbmJfdTyAUnbkAAFLdk23Zbg817.jpg
```

![fdfs集群测试图片1](fdfs集群测试图片1.jpg)

![fdfs集群测试图片2](fdfs集群测试图片二.jpg)

# 4. fastdfs服务端常用命令

fastdfs安装完成后，所有的命令都放在`/usr/bin/`目录下：

```bash
find /usr/bin/ -name fdfs*
```

常用命令有：

```bash
# tracker server相关
fdfs_trackerd /etc/fdfs/tracker.conf start|stop|restart|status
# storage server相关
fdfs_storaged /etc/fdfs/storage.conf start|stop|restart|status
# 在服务器上上传文件
fdfs_upload_file /etc/fdfs/client.conf filepath
# 在服务器上查看文件信息
fdfs_file_info  /etc/fdfs/client.conf file_id
# 在服务器上下载文件
fdfs_download_file /etc/fdfs/client.conf file_id local_filename
# 在服务器上删除文件
fdfs_delete_file /etc/fdfs/client.conf file_id
# 查看集群状态
fdfs_monitor /etc/fdfs/storage.conf
# 删除group中的节点（被删除的节点需要先停机，删除后重启tracker server即可看到最新的集群状态）
fdfs_monitor /etc/fdfs/storage.conf delete group3 192.168.5.121
```

# 5. java客户端集成fastdfs

## 5.1. 构建jar包

​		由于maven中央仓库找不到官方提供的maven坐标，所以需要根据源码打包并放到本地maven仓库。

```http
# 源码下载地址
https://github.com/happyfish100/fastdfs-client-java
```

​		将源码解压到本地后，在代码根目录下打开`cmd`命令窗口，执行`mvn clean install`后，jar包即导入本地仓库，之后将该jar包引入到项目中。

```xml
<dependency>
    <groupId>org.csource</groupId>
    <artifactId>fastdfs-client-java</artifactId>
    <version>1.29-SNAPSHOT</version>
</dependency>
```

## 5.2. 添加配置文件

​		复制源代码中`fastdfs-client-java-master/src/main/resources/fdfs_client.conf.sample`文件为`fdfs_client.conf`，放在项目的`resource`目录下，并修改`tracker_server`配置。

```properties
connect_timeout = 5
network_timeout = 30
charset = UTF-8
http.tracker_http_port = 8080
http.anti_steal_token = true
http.secret_key = 1234567

tracker_server = xxx:22122
tracker_server = xxx:22122

connection_pool.enabled = true
connection_pool.max_count_per_entry = 500
connection_pool.max_idle_time = 3600
connection_pool.max_wait_time_in_ms = 1000
```

## 5.3. 添加工具类

fastdfs客户端主要通过`StorageClient`来操作文件，所以添加一个工具类，用来获取该对象。

`StorageClient`和`StorageClient1`逻辑一致，区别为`StorageClient`将`group name`和`file path`分开处理，而`StorageClient1`将这两个参数统一处理；

**注意：fastdfs java client 的几个主要类都不是线程安全的，每次使用前需要新建一个client。**

``` java
import org.csource.fastdfs.*;

public class FastDFSUtil {
    
    static {
        ClientGlobal.init("fdfs_client.conf");
    }

    public static StorageClient getStorageClient() {

		// return new StorageClient1();
        return new StorageClient();
    }
}
```

## 5.4. 上传文件

``` java
import com.lhx.fastdfs.util.FastDFSUtil;
import org.csource.fastdfs.*;

public class UploadFile {


    public static void main(String[] args) throws Exception {

        StorageClient storageClient = FastDFSUtil.getStorageClient();

        // 返回group name和新创建的file name
        String[] fileInfos = storageClient.upload_file("C:\\Users\\Administrator\\Desktop\\1.jpg",
                "jpg", null);

        for (String fileInfo : fileInfos) {
            System.out.println(fileInfo);
        }

        fileInfos = storageClient.upload_file("C:\\Users\\Administrator\\Desktop\\2.jpg",
                "jpg", null);

        for (String fileInfo : fileInfos) {
            System.out.println(fileInfo);
        }
    }

}
```

![上传文件](上传文件.jpg)

## 5.5. 下载文件

```java
import com.lhx.fastdfs.util.FastDFSUtil;
import org.csource.fastdfs.StorageClient;

public class DownLoadFile {

    public static void main(String[] args) throws Exception{
        StorageClient storageClient = FastDFSUtil.getStorageClient();
        // 这里的group_name和remote_filename为上传文件时方法的返回值
        storageClient.download_file("group1",
                "M00/00/00/wKgFb2Jfs_qALeDaAABiS2pA_yI273.jpg",
                "C:\\Users\\Administrator\\Desktop\\111.jpg");

        storageClient.download_file("group1",
                "M00/00/00/wKgFb2Jfs_qAK7XhAAAtcV-A8_g440.jpg",
                "C:\\Users\\Administrator\\Desktop\\222.jpg");
    }

}
```

## 5.6. 删除文件

``` java
import com.lhx.fastdfs.util.FastDFSUtil;
import org.csource.fastdfs.StorageClient;

public class DeleteFile {

    public static void main(String[] args) throws Exception {

        StorageClient storageClient = FastDFSUtil.getStorageClient();
        // 这里的group_name和remote_filename为上传文件时方法的返回值
        storageClient.delete_file("group1", 
                "M00/00/00/wKgFb2Jfs_qALeDaAABiS2pA_yI273.jpg");

    }
    
}
```

# 6. http访问资源及防盗链

## 6.1 浏览器直接请求资源

fastdfs推荐使用http请求的方式直接下载资源，方式为：`http://storageServerIp:httpServerPort`+`fileId`，请求路径中各字段的含义如下：

- **storageServerIP**：storage server的ip地址；
- **httpServerPort**：storage server开放的http端口号，在`/etc/fdfs/storage.conf`中的`http.server_port`上配置；
- **fileId**：文件id，上传文件时由storage server生成；

通过这种方式，在得知storage server ip和端口信息，以及文件id的时候，可随时获取该资源，显然这种方式不太安全，可使用防盗链的方式解决。

## 6.2. 防盗链

`fastdfs-nginx-module`提供了`token`方式的防盗链（默认是关闭的），在请求的url后跟上`token`和`ts`两个参数即可：

- **ts**：生成token的时间，单位为秒（unix时间戳）；
- **token**：32位的token字符串（md5签名）；

### 6.2.1. 服务端开启防盗链

**<font color="red">注意：集群中各节点的时间一定要一致，否则使用防盗链时，可能会出现token失效时间不一致的情况！！！</font>**

修改storage server上`/etc/fdfs/http.conf`配置，并重启`nginx`，示例配置如下：

```bash
# HTTP default content type
http.default_content_type = application/octet-stream

# MIME types mapping filename
# MIME types file format: MIME_type  extensions
# such as:  image/jpeg	jpeg jpg jpe
# you can use apache's MIME file: mime.types
http.mime_types_filename = mime.types

# 开启token防盗链
http.anti_steal.check_token = true

# token失效时间，单位为秒
http.anti_steal.token_ttl = 120

# 生成token的私有key，与客户端配置文件中保持一致
http.anti_steal.secret_key = FastDFS1234567890

# token失效时展示的图片
http.anti_steal.token_check_fail = /home/yuqing/fastdfs/conf/anti-steal.jpg
http.multi_range.enabed = true
```

### 6.2.2. 客户端开启防盗链

java客户端中修改`fdfs_client.conf`配置，示例如下：

```bash
connect_timeout = 5
network_timeout = 30
charset = UTF-8
http.tracker_http_port = 8080
# 开启防盗链
http.anti_steal_token = true
# 生成token的私有key，与服务端保持一致
http.secret_key = FastDFS1234567890

tracker_server = 192.168.5.112:22122

connection_pool.enabled = true
connection_pool.max_count_per_entry = 500
connection_pool.max_idle_time = 3600
connection_pool.max_wait_time_in_ms = 1000
```

### 6.2.3. 生成token

使用客户端SDK自带的方法生成token

```java
package com.lhx.fastdfs;

import org.csource.common.MyException;
import org.csource.fastdfs.ProtoCommon;

import java.io.UnsupportedEncodingException;
import java.security.NoSuchAlgorithmException;

public class AntiSteal {

    public static void main(String[] args) throws UnsupportedEncodingException, NoSuchAlgorithmException, MyException {
        String remoteFileName = "M00/00/00/wKgFbmJlCe2AFIY2ABTNkfy_PF0798.mp4";
        int ts = (int)(System.currentTimeMillis() / 1000);
        System.out.println(ts);
        String secretKey = "FastDFS1234567890";
        String token = ProtoCommon.getToken(remoteFileName, ts, secretKey);
        System.out.println(token);
    }
}
```

### 6.2.4. 测试防盗链效果

上传一个测试文件：

```bash
# 先上传一个图片素材到storage server
fdfs_upload_file /etc/fdfs/client.conf /usr/local/11.jpg
# 得到文件id
group1/M00/00/00/wKgFb2JlB6OAQzViAAFLdk23Zbg767.jpg 
```

直接在浏览器访问上传的文件，提示没有权限：

```http
http://192.168.5.112/group1/M00/00/00/wKgFb2JlB6OAQzViAAFLdk23Zbg767.jpg
```

![找不到资源](找不到资源.jpg)

生成token后再访问：

```http
http://192.168.5.112/group1/M00/00/00/wKgFb2JlB6OAQzViAAFLdk23Zbg767.jpg?token=d25ae21f9a18d8831b0639ea3a488ed8&ts=1650791311
```

![使用防盗链](使用防盗链.jpg)

token过期后再访问：

![token过时](token过时.jpg)

# 7. fastdfs集群扩容

## 7.1. 横向扩容

​		横向扩容指增加`storage cluster`中group的数量，例如现有集群架构为2台tracker server组成tracker cluster，4台storage server组成storage cluster，且两两分为group1和group2，现在要对集群进行横向扩容，即storage cluster新增加由两台storage server组成的group3，则需要以下几个步骤：

- 配置好两台新增storage server的单机环境，启动storage server即可；
- 修改所有tracker server上nginx的负载配置，把新增的storage server地址加上去，并重启nginx；

​		**综上，横向扩容的方法在只用重启tracker server上的nginx，鉴于生产环境架构为tracker cluster，所以该方式可逐台重启tracker server上的nginx，即可以在不停服务的情况下完成动态扩容。**

## 7.2. 纵向扩容

​		纵向扩容指对group的容量进行扩容，因为同一group内机器的数据是互相备份的，故同一个group的最大容量取决于group内最小strorage的存储容量，若要对一个group进行纵向扩容，则需要以下几个步骤：

- 对group内所有机器挂载相同容量的磁盘；
- 修改group内所有机器`storage.conf`和`mod_fastdfs.conf`配置文件中的`store_path_count`属性，并添加相应的`store_path`，最后再重启storage server和nginx即可；

**<font color="red">注意：修改完单台storage server后，该节点会切换到OFFLINE状态（下线状态，不可上传，可通过浏览器访问），直到group内所有节点调整完毕后，才会切换到ACTIVE状态。</font>**

​		**综上，纵向扩容的方式需要逐台重启group内的storage server和nginx，待group内所有节点重启完毕后，该group方可提供服务，即这种扩容方式需要暂停group的服务（其他group无影响，该group仅可通过浏览器访问已上传的文件），时间大概为秒级（最后一台storage server重启的时间 + fastdfs cluster调整group storage count的时间）。**
