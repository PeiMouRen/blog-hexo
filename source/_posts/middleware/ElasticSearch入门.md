---
title: ElasticSearch入门
date: 2022-08-31 09:38:13
categories:
  - ElasticSearch
tags:	
  - ES
  - ElasticSearch
---

# 1. 简介

> **官网描述：**
>
> ​		Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。
>
> **个人理解：**
>
> ​		**NoSQL数据库**和**搜索引擎**的结合体，底层基于`Lucence`(全文搜索引擎)，采用`Java`开发，屏蔽了底层`Lucence`复杂的使用方式，提供了一套简洁的`RESTful API`。

官网地址：`https://www.elastic.co/cn/elasticsearch/`

下载地址：`https://www.elastic.co/cn/downloads/elasticsearch`

中文文档（版本比较旧）：`https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html`

## 基础概念

- **Near Realtime（NRT）** 近实时。数据提交索引后，立马就可以搜索到。

- **Cluster 集群**，一个集群由一个唯一的名字标识，默认为“elasticsearch”。集群名称非常重要，具有相同集群名的节点才会组成一个集群。集群名称可以在配置文件中指定。

- **Node 节点**：存储集群的数据，参与集群的索引和搜索功能。像集群有名字，节点也有自己的名称，默认在启动时会以一个随机的UUID的前七个字符作为节点的名字，你可以为其指定任意的名字。通过集群名在网络中发现同伴组成集群。一个节点也可是集群。

- **Index 索引**: 一个索引是一个文档的集合（等同于solr中的集合）。每个索引有唯一的名字，通过这个名字来操作它。一个集群中可以有任意多个索引。

- **Type 类型**：指在一个索引中，可以索引不同类型的文档，如用户数据、博客数据。从6.0.0 版本起已废弃，一个索引中只存放一类数据。

- **Document 文档**：被索引的一条数据，索引的基本信息单元，以JSON格式来表示。

- **Shard 分片**：在创建一个索引时可以指定分成多少个分片来存储。每个分片本身也是一个功能完善且独立的“索引”，可以被放置在集群的任意节点上。

- **Replication 备份**: 一个分片可以有多个备份（副本）。

---

**关系型数据库与es对比：**

|         RDBMS          |       ElasticSearch        |
| :--------------------: | :------------------------: |
|    数据库(database)    |        索引(index)         |
|       表(table)        | 类型(type)(新版本中已废弃) |
|        行(row)         |       文档(document)       |
|       列(column)       |        字段(field)         |
|     表结构(schema)     |       映射(mapping)        |
|    索引（正向索引）    |    反向索引（倒排索引）    |
|          SQL           |          查询DSL           |
| SELECT * FROM TABLE... |       GET http://...       |
|  UPDATE TABLE SET...   |       PUT http://...       |
|  DELETE FROM TABLE...  |     DELETE http://...      |

## RESTful请求类型

- **GET**：查询
- **POST**：新建
- **PUT**：更新
- **DELETE**：删除

# 2. 安装

> 可使用安装包安装，本文档使用Docker，版本为8.4.1

```bash
# 拉取镜像
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.4.1
```

# 3. 启动

## 3.1.  单节点集群启动

```bash
# 创建docker network
docker network create elastic
# 启动es容器，启动成功后会在终端输出elastic用户的密码、配置集群的token、Kibana的注册token等信息
# 这些信息仅在第一次启动es时显示，请妥善保存这些信息
docker run -it --name es01 --net elastic -p 9200:9200 -p 9300:9300 \
docker.elastic.co/elasticsearch/elasticsearch:8.4.1

docker run -it --name es01 --net es -p 9200:9200 -p 9300:9300 \
-v /tmp/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
docker.elastic.co/elasticsearch/elasticsearch:8.4.1

# 重新开启一个终端
# 将生成的ca证书复制到当前目录
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
# 访问es服务器，返回json串
curl --cacert http_ca.crt -u elastic https://localhost:9200
```

![es启动成功](es启动成功.jpg)

### 3.1.1.  Docker启动失败

> **报错：**max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

**解决方法：**

```bash
# 修改值
sysctl -w vm.max_map_count=262144
# 查看修改后的值
sysctl -a | grep vm.max_map_count
# 重新加载
sysctl -p
```

## 3.2. 向单节点集群中加入新节点

**注意：**3.1.步启动单节点集群后，生成的`token`在其他节点加入该集群时使用，有效期为30分钟，失效后需要重新生成。

**生成token：**

```bash
# 进入容器
docker exec -it es01 /bin/bash
# 生成新token
./bin/elasticsearch-create-enrollment-token -s node
```

**启动新节点并加入集群：**

```bash
# 将token替换为自己的
docker run -it -e ENROLLMENT_TOKEN="token" --name es02 --net elastic  docker.elastic.co/elasticsearch/elasticsearch:8.4.1

```

## 3.3. 用Docker Compose启动多节点集群

参考官方文档`https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-compose-file`

# Kibana

## 简介

Kibana与es配合使用，用于可视化es的数据，并附带es的开发工具。

## 安装

```bash
# 拉取kibana镜像
docker pull docker.elastic.co/kibana/kibana:8.4.1
# 启动kibana容器
docker run -d --name kib01 --net elastic -p 5601:5601 \
-v /usr/share/kibana/config:/usr/share/kibana/config \
docker.elastic.co/kibana/kibana:8.4.1
# 查看kibana访问连接
docker logs kib01
```

## 设置中文

配置文件`kibana.yml`中添加`i18n.locale: zh-CN`

## es重新生成kibana连接用的token

`./bin/elasticsearch-create-enrollment-token -s kibana`

# es命令

## 索引

### 创建索引

```
PUT /myindex02	// 索引名称
{
  "settings": {	// 索引设置配置
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": { // 索引映射配置
    "properties": {
      "id": {
        "type": "integer"
      },
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      }
    }
  }
}
```

### 查找索引

```
GET /myindex02 // 查找整个索引信息
GET /myindex02/_settings // 查找索引设置信息
GET /myindex02/_mapping // 查找索引映射信息
```

### 删除索引

```
DELETE /myindex01
```

## 映射

映射是定义文档及其包含的字段如何存储和索引的过程，分为**动态映射**和**显示映射**。

### 动态映射

即由es自动创建index，并自动发现index的映射。

```
PUT /data/_doc/1
{
  "count": 5
}

GET /data
```

### 显示映射

即创建index时显示指定映射。

```
PUT /myindex001
{
  "mappings": {
    "properties": {
      "age": { "type": "integer" },
      "email": { "type": "keyword"},
      "name": { "type": "text"}
    }
  }
}

GET /myindex001/_mapping
```

### 字段数据类型

包括Binary、Boolean、Date、Keyword、Text、Range等类型。









## 文档

### 添加/更新文档

```
POST /myindex02/_doc/1 // 指定文档id为1，不存在则添加，存在则更新
{
  "id": 1,
  "name": "zhangsan",
  "age" : 20
}

POST /myindex02/_doc // 自动生成文档id，不存在则添加，存在则更新
{
  "id": 2,
  "name": "lisi",
  "age" : 21
}
```

### 获取文档

```
// 指定文档id
GET /myindex02/_doc/1

// 查询张三的信息
GET /myindex02/_search?from=0&size=2
{
  "query": {
    "term": {
      "name": "zhangsan"
    }
  }
}
```

### 删除文档

```
DELETE /myindex02/_doc/1 // 指定文档id
```

## 查询

### 组合查询

#### bool查询

bool即布尔，用作组合多个查询条件。

---

**关键字**

- must: 必须匹配，参与计算评分；
- filter：过滤查询结果，不参与计算评分；
- should：选择性匹配，即必须满足其中一个条件，参与计算评分；
- must_not：必须不匹配，不参与计算评分；

---

**查询示例**

```
GET /myindex02/_search
{
  "from": 0,
  "size": 20,
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "name": "zhangsan"
          }
        }
      ],
      "should": [
        {
          "term": {
            "age": 21
          }
        },
        {
          "term": {
            "age": 22
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "id": {
              "gte": 10,
              "lte": 20
            }
          }
        }
      ]
    }
  }
}
```

### 术语查询

**exists**

查询给定字段是否存在；

```
GET /myindex02/_search
{
  "query": {
    "exists": {
      "field": "name"
    }
  }
}
```

---

**fuzzy**

即模糊查询，查询场景如下：

- 改变一个字符（**b** ox → **f** ox）
- 删除一个字符（**b**缺少 → 缺少）
- 插入一个字符 (sic → sic **k** )
- 转置两个相邻字符 ( **ac** t → **ca** t)

```
GET /myindex02/_search
{
  "query": {
    "fuzzy": {
      "name": "zhangan"
    }
  }
}
```

---

**ids**

通过文档id集合查询；

```
GET /myindex02/_search
{
  "query": {
    "ids": {
      "values": [1, 2, 3]
    }
  }
}
```

---

**prefix**

前缀查询；

```
GET /myindex02/_search
{
  "query": {
    "prefix": {
      "name": "zhang"
    }
  }
}
```

---

**range**

范围查询；

```
GET /myindex02/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 30
      }
    }
  }
}
```

---

**term**

精确查询；

```
GET /myindex02/_search
{
  "query": {
    "term": {
      "name": "zhangsan"
    }
  }
}
```

---

**terms**

增强term查询；

```
GET /myindex02/_search
{
  "query": {
    "terms": {
      "name": ["zhangsan", "lisi", "wangwu"]
    }
  }
}
```

---

**wildcard**

通配符查询；

```
GET /myindex02/_search
{
  "query": {
    "wildcard": {
      "name": "zhan*"
    }
  }
}
```

### 聚合查询

#### 桶聚合

**terms桶聚合**

```
GET /myindex02/_search
{
  "size": 0, 
  "aggs": {
    "myaggs": {
        "terms": {
          "field": "age"
        }
    }
  }
}
```

---

**composite桶聚合**

```
GET /myindex02/_search
{
  "aggs": {
    "aggs1": {
      "composite": {
        "sources": [
          {
            "a1": {
              "terms": {
                "field": "age"
              }
            }
          },
          {
            "a2": {
              "terms": {
                "field": "id"
              }
            }
          }
        ]
      }
    }
  }
}
```

---

**嵌套桶聚合**

```
GET /myindex02/_search
{
  "aggs": {
    "a1": {
      "terms": {
        "field": "age"
      },
      "aggs": {
        "a2": {
          "avg": {
            "field": "id"
          }
        }
      }
    }
  }
}
```

#### 指标聚合

**最大值**

```
GET /myindex02/_search
{
  "aggs": {
    "a1": {
      "max": {
        "field": "age"
      }
    }
  }
}
```

### curl方式查询

```bash
# -d表示json数据，json不能换行，且需要用单引号括起来
curl -X GET "http://localhost:9200/hostmsg-get-2022.10.08/_search?pretty" -u elastic:YZ*psc521 -H 'Content-Type: application/json' -d ' 
{	"from": 0, "size":1, "query": {	"match": {	"msg": "000001489810016120006221881*"	}}	} '
```



# es-client插件

Firefox
