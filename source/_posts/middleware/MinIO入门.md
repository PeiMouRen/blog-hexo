---
title: MinIO入门
date: 2022-08-16 15:18:53
categories: 
  - 中间件
  - 分布式文件存储
tags:
  - MinIO	
  - 分布式文件存储
---

[TOC]

# 1. 单机环境

## 1.1. 版本

```
Linux:CentOS7
```

## 1.2. 步骤

```bash
# 检查是否有wget
wget -V
# 没有的话安装一下
yum -y install wget

# 下载资源（不同架构的系统有不同的url）
wget https://dl.min.io/server/minio/release/linux-amd64/minio
# 添加执行权限
chmod +x minio
# 启动minio server，/data为自定义存储minio数据的目录
./minio server /data
```

## 1.3. 验证

浏览器访问`http://192.168.5.112:9000 `，使用root账号`minioadmin/minioadmin`登录成功即可。

**浏览器访问不到的话，检查服务器是否开放端口。**

# 2. 集群环境

## 2.1. 参考资料

```http
# MinIO纠删码入门
https://docs.min.io/docs/minio-erasure-code-quickstart-guide.html
# MinIO分布式入门
https://docs.min.io/docs/distributed-minio-quickstart-guide.html
```

## 2.2. 搭建要点 

- 分布式Minio采用 [纠删码](http://docs.minio.org.cn/docs/master/minio-erasure-code-quickstart-guide)来防范多个节点宕机和[位衰减`bit rot`](https://github.com/minio/minio/blob/master/docs/zh_CN/erasure/README.md#what-is-bit-rot-protection)。
- 使用分布式Minio自动引入了纠删码功能。
- MinIO 将提供的硬盘划分为*4 到 16 个*硬盘的纠删码集，因此，搭建集群提供的硬盘数量必须是这些数字之一的倍数，每个对象都被写入一个单独的纠删码集。
- Minio 使用最大可能的 纠删码集大小，它分为给定的硬盘数量。例如，*18 个硬盘*配置为*2 组 9 个硬盘*，*24 个硬盘*配置为*2 组 12 个硬盘*。
- 建议运行分布式 MinIO 设置的所有节点是同质的，即相同的操作系统、相同数量的磁盘和相同的网络互连。
- 单机Minio服务存在单点故障，相反，如果是一个有N块硬盘的分布式Minio,只要有N/2硬盘在线，你的数据就是安全的。不过你需要至少有N/2+1个硬盘来创建新的对象。
- Minio在分布式和单机模式下，所有读写操作都严格遵守**read-after-write**一致性模型。
- 分布式Minio里的节点时间差不能超过15分钟，你可以使用[NTP](http://www.ntp.org/) 来保证时间一致。
- 分布式Minio使用的磁盘可以与其他程序共享，但是要新建一个目录用来存放MinIO的数据。
- 所有运行分布式 MinIO 的节点都应该共享一个共同的根凭证，以便节点相互连接和信任。为此，建议在执行 MinIO 服务器命令之前，在所有节点上将`MINIO_ROOT_USER`和 `MINIO_ROOT_PASSWORD`导出为环境变量，如果未导出，可使用默认用户`minioadmin/minioadmin`。

## 2.3. 步骤

本次启动MinIO集群实例为四个节点，每个节点一块硬盘，在四个节点上分别执行如下命令：

```bash
# /data/minio是自定义的存储路径
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
./minio server http://192.168.5.110/data/minio \
			   http://192.168.5.111/data/minio \
			   http://192.168.5.112/data/minio \
			   http://192.168.5.121/data/minio
```

## 2.4. 验证

浏览器访问任意服务器的`9000`端口，使用root账号`admin/12345678`登录，查看集群状态。

![minio集群](minio集群.jpg)

# 3. java客户端集成

## 3.1. 引入依赖

```xml
<!-- https://mvnrepository.com/artifact/io.minio/minio -->
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.3.9</version>
</dependency>
```

## 3.2. 上传文件

```java
import io.minio.*;
import java.io.File;
import java.io.FileInputStream;

public class FileUploader {

    public static void main(String[] args) throws Exception {
        String endPoint = "http://192.168.5.112:9000";
        String rootUser = "admin";
        String rootPassword = "12345678";
        String bucketName = "first-bucket";

        // 构建客户端对象
        MinioClient minioClient = MinioClient
                .builder()
                .endpoint(endPoint)
                .credentials(rootUser, rootPassword)
                .build();

        // 创建bucket
        boolean found = minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
        if (found) {
            System.out.println(bucketName + " exists");
        } else {
            minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
            System.out.println(bucketName + " created");
        }

        // 上传文件

        // 通过文件流上传
        File file = new File("C:\\Users\\Administrator\\Desktop\\刘德华 - 暗里着迷 (Live).mp3");
        long fileSize = file.length();
        System.out.println("file size: " + fileSize);
        FileInputStream fileInputStream = new FileInputStream(file);
        ObjectWriteResponse objectWriteResponse = minioClient.putObject(PutObjectArgs.builder()
                .bucket(bucketName)
                .object("/music/anlizhaomi.mp3")
                .stream(fileInputStream, fileSize, -1)
                .contentType("audio/mp3")
                .build());
        fileInputStream.close();
        System.out.println("上传结果：" + objectWriteResponse.etag() + " " + objectWriteResponse.versionId());

        // 通过文件上传
        objectWriteResponse = minioClient.uploadObject(UploadObjectArgs.builder()
                .bucket(bucketName)
                .object("/music/anlizhaomi_1.mp3")
                .filename("C:\\Users\\Administrator\\Desktop\\刘德华 - 暗里着迷 (Live).mp3")
                .contentType("audio/mp3")
                .build());
        System.out.println("上传结果：" + objectWriteResponse.etag() + " " + objectWriteResponse.versionId());
    }
}
```

## 3.3. 下载文件

```java
import io.minio.DownloadObjectArgs;
import io.minio.GetObjectArgs;
import io.minio.GetObjectResponse;
import io.minio.MinioClient;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;

public class FileDownLoader {

    public static void main(String[] args) throws Exception {

        String endPoint = "http://192.168.5.112:9000";
        String rootUser = "admin";
        String rootPassword = "12345678";
        String bucketName = "first-bucket";

        // 构建客户端对象
        MinioClient minioClient = MinioClient
                .builder()
                .endpoint(endPoint)
                .credentials(rootUser, rootPassword)
                .build();

        // 下载文件

        // 通过流下载
        GetObjectResponse stream = minioClient.getObject(GetObjectArgs
                .builder()
                .bucket(bucketName)
                .object("/music/anlizhaomi.mp3")
                .build());
        BufferedInputStream bufferedInputStream = new BufferedInputStream(stream);
        FileOutputStream fileOutputStream = new FileOutputStream("C:\\Users\\Administrator\\Desktop\\aa.mp3");
        BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
        byte[] bytes = new byte[1024];
        int length;
        while ((length = bufferedInputStream.read(bytes)) != -1) {
            bufferedOutputStream.write(bytes, 0, length);
        }
        bufferedInputStream.close();
        stream.close();
        bufferedOutputStream.close();
        fileOutputStream.close();

        // 通过文件下载
        minioClient.downloadObject(DownloadObjectArgs
                .builder()
                .bucket(bucketName)
                .object("/music/anlizhaomi_1.mp3")
                .filename("C:\\Users\\Administrator\\Desktop\\bb.mp3")
                .build());

    }

}
```

## 3.3. 删除文件

```java
import io.minio.MinioClient;
import io.minio.RemoveObjectArgs;

public class FileDeleter {

    public static void main(String[] args) throws Exception {

        String endPoint = "http://192.168.5.112:9000";
        String rootUser = "admin";
        String rootPassword = "12345678";
        String bucketName = "first-bucket";

        // 构建客户端对象
        MinioClient minioClient = MinioClient
                .builder()
                .endpoint(endPoint)
                .credentials(rootUser, rootPassword)
                .build();

        // 删除文件
        minioClient.removeObject(RemoveObjectArgs
                .builder()
                .bucket(bucketName)
                .object("/music/anlizhaomi.mp3")
                .build());
    }

}
```

## 3.4. 浏览器预览文件

```java
import io.minio.GetPresignedObjectUrlArgs;
import io.minio.MinioClient;
import io.minio.http.Method;

import java.util.concurrent.TimeUnit;

public class FilePreviewUrl {

    public static void main(String[] args) throws Exception {

        String endPoint = "http://192.168.5.112:9000";
        String rootUser = "admin";
        String rootPassword = "12345678";
        String bucketName = "first-bucket";

        // 构建客户端对象
        MinioClient minioClient = MinioClient
                .builder()
                .endpoint(endPoint)
                .credentials(rootUser, rootPassword)
                .build();
		// 直接拿这个url到浏览器访问即可
        String url = minioClient.getPresignedObjectUrl(GetPresignedObjectUrlArgs
                .builder()
                .method(Method.GET)
                .bucket(bucketName)
                .object("/music/浏览器下载文件.jpg")
                .expiry(2, TimeUnit.MINUTES)
                .build());
        System.out.println("url: " + url);

    }

}
```

# 4. 与FastDFS比较

- 同为开源项目，MinIO社区较活跃。
- MinIO有官网、详细的文档及主流开发语言的SDK。
- MinIO读/写速度上可达183 GB / 秒 和 171 GB / 秒。
- MinIO有自带的web控制台界面。
- MinIO安装部署比较简单。
- MinIO可与云原生应用高度集成。
