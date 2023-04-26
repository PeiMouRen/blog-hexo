---
title: ShardingSphere入门
date: 2022-08-16 16:13:03
categories:
  - 中间件
  - 分库分表
tags:
  -	ShardingSphere
  -	分库分表
---

[toc]

# 简介

> shardingsphere是一款apache开源的分布式数据库生态项目，可提供数据分片、读写分离、数据加密等功能，由JDBC、Proxy、Sidecar三个模块组成，每个模块可独立部署；
>
> - shardingsphere-JDBC：部署在客户端，通过jar包引入，可与java项目快速集成；
> - shardingsphere-Proxy：部署在服务端，可以向连接数据库一样连接Proxy；
> - shardingsphere-Sidecar：云原生数据库代理，目前还处于规划阶段；
>
> 官方网站：`https://shardingsphere.apache.org/index_zh.html`
>
> github:`https://github.com/apache/shardingsphere`

# 使用

> 以shardingsphere-proxy + postgresql + zookeeper集群为例，配置数据分片 + 读写分离
>
> 版本：
>
> ​	shardingsphere-proxy:5.1.2
>
> ​	zookeeper:3.8.0
>
> ​	jdk:17
>
> ​	postgresql:12.6

## 规划

三个节点的zookeeper集群和三个节点的proxy集群；

四个pg库，两两一组做读写分离；

## 准备

服务器四台：172.19.3.60/61/62/63

60、61、62三台服务器分别配置jdk、zookeeper、proxy环境；

60、61、62、63分别配置好pg环境，且61为60的从机，63为62的从机；

省略jdk配置、zookeeper集群搭建、pg安装步骤；

---

**两台pg主机（60、62）上建库建表**

```sql
# 建库
./createdb shard1
# 在shard1上建表
create table t_user_0(
	id bigint,
    name varchar(200),
    phone varchar(11)
);
create table t_user_1(
	id bigint,
    name varchar(200),
    phone varchar(11)
);
```

## 下载安装proxy

**下载地址**

```http
https://shardingsphere.apache.org/document/current/cn/downloads/
```

**安装**

解压二进制包即可

## 配置proxy

**60服务器proxy配置**

`conf/server.yaml`

```yaml
# 采用zookeeper的集群模式
mode:
  type: Cluster
  repository:
    type: ZooKeeper
    props:
      namespace: governance_ds
      server-lists: 172.19.3.60:2181,172.19.3.61:2181,172.19.3.62:2181
      retryIntervalMilliseconds: 500
      timeToLiveSeconds: 60
      maxRetries: 3
      operationTimeoutMilliseconds: 500
  overwrite: true # 表示本地配置会覆盖zookeeper中的配置 
# 权限配置
rules:
  - !AUTHORITY
    users: # 配置连接proxy的用户，该用户必须在服务器上存在
      - root@%:123
      - postgres@%:postgres
      - sms@%:itc123
    provider:
      type: ALL_PRIVILEGES_PERMITTED

props: 
  sql-show: true # 在日志中显示sql语句
```

`conf/config-sharding.yaml`

```yaml
databaseName: smsdb # proxy中的逻辑数据库名称
# 数据源配置
dataSources:
  ds0_master: # 自定义数据源名称
    url: jdbc:postgresql://172.19.3.60:5432/shard1?stringtype=unspecified
    username: sms
    password: itc123
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    minPoolSize: 1
  ds0_slave:
    url: jdbc:postgresql://172.19.3.61:5432/shard1?stringtype=unspecified
    username: sms
    password: itc123
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    minPoolSize: 1
  ds1_master:
    url: jdbc:postgresql://172.19.3.62:5432/shard1?stringtype=unspecified
    username: sms
    password: itc123
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    minPoolSize: 1
  ds1_slave:
    url: jdbc:postgresql://172.19.3.63:5432/shard1?stringtype=unspecified
    username: sms
    password: itc123
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    minPoolSize: 1
  
# 规则配置
rules:
# 分片规则配置
- !SHARDING
  tables:
    t_user: # 自定义逻辑表名
      # 真实节点，使用行表达式，该配置的含义是：t_user表被分成四份，坐标为
      # readwrite_ds0.t_user_0、readwrite_ds0.t_user_1
      # readwrite_ds1.t_user_0、readwrite_ds1.t_user_1
      actualDataNodes: readwrite_ds$->{0..1}.t_user_$->{0..1}
      # 分库策略
      databaseStrategy: 
        standard: 
          shardingColumn: id # 分库依据列
          shardingAlgorithmName: t_user_inline # 分库算法名，需在shardingAlgorithms下定义
      # 分表策略
      tableStrategy:
        standard:
          shardingColumn: phone # 分表依据列
          shardingAlgorithmName: t_user_custom # 分表算法名，需在shardingAlgorithms下定义
      # 主键生成策略绝
      keyGenerateStrategy:
        column: id # 主键列
        keyGeneratorName: snowflake # 主键生成算法，需在keyGenerators下定义
  # 广播表配置
  broadcastTables:
    - t_city
  # 分片算法配置
  shardingAlgorithms:
    t_user_custom: # 自定义算法名
      type: CLASS_BASED # 使用自定义的算法
      props:
        strategy: STANDARD
        # 自定义算法类全路径，这个算法是根据手机尾号对2取模
        algorithmClassName: com.xx.sharding.algorithm.PhoneTailShardingAlgorithm
    t_user_inline:
      type: INLINE # 使用行表达式
      props:
        algorithm-expression: readwrite_ds${id % 2}
  # 主键生成算法
  keyGenerators:
    snowflake: # 自定义算法名
      type: SNOWFLAKE # 使用内置的雪花算法
# 读写分离规则
- !READWRITE_SPLITTING
  # 数据源配置
  dataSources:
    readwrite_ds0: # 自定义数据源名称
      type: Static # 静态路由
      props:
        write-data-source-name: ds0_master # 写库，数据源需在dataSources下定义
        read-data-source-names: ds0_slave # 读库，数据源需在dataSources下定义
      loadBalancerName: random # 这里指定的算法名需在loadBalancers下定义
    readwrite_ds1:
      type: Static
      props:
        write-data-source-name: ds1_master
        read-data-source-names: ds1_slave
      loadBalancerName: random
  # 负载均衡算法
  loadBalancers:
    random: # 自定义算法名称
      type: RANDOM # 使用内置的随机算法
```

---

**61、62服务器proxy配置**

`conf/server.yaml`

```yaml
# 采用zookeeper的集群模式
mode:
  type: Cluster
  repository:
    type: ZooKeeper
    props:
      namespace: governance_ds
      server-lists: 172.19.3.60:2181,172.19.3.61:2181,172.19.3.62:2181
      retryIntervalMilliseconds: 500
      timeToLiveSeconds: 60
      maxRetries: 3
      operationTimeoutMilliseconds: 500
  overwrite: false # 表示使用zookeeper中的配置 
```

## 启动proxy

```bash
bin/start.sh
```

## 连接proxy

### 使用客户端连接

```bash
# psql：使用pg客户端连接	-h:proxy所在地址	-p:proxy的端口，默认为3307	
# -U：登录用户，在server.yaml中配置	-d:连接数据库，在config-sharding.yaml中配置
psql -h 172.19.3.60 -p 3307 -U root -d smsdb
```

### 使用工具连接

> 这里使用navicat16测试可连接成功

![navicat连接proxy](navicat连接proxy.png)

### jdbc连接

使用pg的驱动连接即可，例如`Spring Boot`数据源配置：

```yaml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://172.19.3.60:3307/smsdb
    username: root
    password: 123
```

## 验证

> 这里用navicat连接proxy做一些简单验证；
>
> 说明：上面proxy分片和读写分离的策略为：根据id%2分库，根据phone尾号%2分表，ds0_master、ds1_master为主库，可以插入数据，ds0_slave、ds1_slave为从库，只可以读取数据。

### 插入数据

**sql**:

```sql
# 这里表名前需要加数据库中真实表表所在的schema，否则生成主键失败
INSERT INTO sms.t_user(NAME, PHONE) VALUES('zhangsan', '18328442123');
```

**分析**：

id是使用雪花算法自动生成的，所以还不确定会插入哪个库；

手机尾号%2=1，所以可以确定该条数据会插入t_user_1表；

**执行并查看日志**：

执行上面的sql，并查看日志`logs/stdout.log`，可以发现，生成的`id`为`761164204387336192`，`id%2=0`，所以会选择读写分离数据源`readwrite_ds0`中可插入数据的数据源`ds0_master`，且插入表为`t_user_1`;

```
[INFO ] 2022-08-02 09:54:48.331 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Logic SQL: INSERT INTO sms.t_user(NAME, PHONE) VALUES('zhangsan', '18328442123')
[INFO ] 2022-08-02 09:54:48.331 [Connection-6-ThreadExecutor] ShardingSphere-SQL - SQLStatement: PostgreSQLInsertStatement(withSegment=Optional.empty)
[INFO ] 2022-08-02 09:54:48.331 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Actual SQL: ds0_master ::: INSERT INTO sms.t_user_1(NAME, PHONE, id) VALUES('zhangsan', '18328442123', 761164204387336192)
```

### 查询数据

**sql**:

```sql
SELECT * FROM sms.t_user;
```

**分析**：

这条sql没有指定分库和分表的查询条件，所以从`ds0_slave`中的`t_user_0`、`t_user_1`和`ds1_slave`中的`t_user_0`、`t_user_1`。

**执行并查看日志**：

```
[INFO ] 2022-08-02 10:02:26.680 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Logic SQL: SELECT * FROM sms.t_user
[INFO ] 2022-08-02 10:02:26.680 [Connection-6-ThreadExecutor] ShardingSphere-SQL - SQLStatement: PostgreSQLSelectStatement(limit=Optional.empty, lock=Optional.empty, window=Optional.empty)
[INFO ] 2022-08-02 10:02:26.680 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Actual SQL: ds0_slave ::: SELECT * FROM sms.t_user_0 UNION ALL SELECT * FROM sms.t_user_1
[INFO ] 2022-08-02 10:02:26.680 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Actual SQL: ds1_slave ::: SELECT * FROM sms.t_user_0 UNION ALL SELECT * FROM sms.t_user_1
```

---

**sql**:

```sql
SELECT * FROM sms.t_user WHERE id = 761164204387336192 AND phone = '18328442123';
```

**分析**：

这条sql指定了分库和分表的查询条件，且`id%2=0`，`phone尾号%2=1`，所以会查询`ds0_slave.t_user_1`。

**执行并查看日志**：

```
[INFO ] 2022-08-02 10:05:48.852 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Logic SQL: SELECT * FROM sms.t_user WHERE id = 761164204387336192 AND phone = '18328442123'
[INFO ] 2022-08-02 10:05:48.853 [Connection-6-ThreadExecutor] ShardingSphere-SQL - SQLStatement: PostgreSQLSelectStatement(limit=Optional.empty, lock=Optional.empty, window=Optional.empty)
[INFO ] 2022-08-02 10:05:48.853 [Connection-6-ThreadExecutor] ShardingSphere-SQL - Actual SQL: ds0_slave ::: SELECT * FROM sms.t_user_1 WHERE id = 761164204387336192 AND phone = '18328442123'
```

## 自定义算法

> shardingsphere使用`SPI`的方式扩展相应算法，下面列举一个自定义的分片算法

1. 实现`StandardShardingAlgorithm`接口；

   ```java
   import com.google.common.collect.Range;
   import org.apache.shardingsphere.sharding.api.sharding.standard.PreciseShardingValue;
   import org.apache.shardingsphere.sharding.api.sharding.standard.RangeShardingValue;
   import org.apache.shardingsphere.sharding.api.sharding.standard.StandardShardingAlgorithm;
   
   import java.sql.Timestamp;
   import java.time.LocalDateTime;
   import java.util.Collection;
   import java.util.LinkedHashSet;
   import java.util.Properties;
   
   /**
    * 根据日期分片
    */
   public class DayShardingAlgorithm implements StandardShardingAlgorithm<String> {
   
       private Properties properties;
   
       @Override
       public void init(Properties props) {
           this.properties = props;
       }
   
       @Override
       public Properties getProps() {
           return this.properties;
       }
   
       /**
       * 精确分片
       **/
       @Override
       public String doSharding(Collection<String> availableTargetNames, PreciseShardingValue<String> shardingValue) {
           int day = getDayFromStr(shardingValue.getValue());
           return shardingValue.getDataNodeInfo().getPrefix() + day;
       }
   
       /**
       * 范围分片
       **/
       @Override
       public Collection<String> doSharding(Collection<String> availableTargetNames, RangeShardingValue<String> shardingValue) {
           Collection<String> tables = new LinkedHashSet<>();
           Range<String> valueRange = shardingValue.getValueRange();
           String prefix = shardingValue.getDataNodeInfo().getPrefix();
           int lowerValue = valueRange.hasLowerBound()? getDayFromStr(valueRange.lowerEndpoint()) : 1;
           int upperValue = valueRange.hasUpperBound()? getDayFromStr(valueRange.upperEndpoint()) : 31;
           for (int i = lowerValue; i <= upperValue; i++) {
               tables.add(prefix + i);
           }
           return tables;
       }
   
       private int getDayFromStr(String dateStr) {
   
           if (dateStr == null) {
               throw new NullPointerException("日期不能为空! ");
           }
   
           int beginIndex = 0;
           int endIndex = 0;
           int length = dateStr.length();
           if (length == 8) {
               beginIndex = 6;
               endIndex = 8;
           } else if (length >= 10) {
               beginIndex = 8;
               endIndex = 10;
           } else {
               throw new IllegalArgumentException("日期格式异常: " + dateStr);
           }
   
           return Integer.parseInt(dateStr.substring(beginIndex, endIndex));
       }
   }
   ```

2. 在项目 `resources` 目录下创建 `META-INF/services` 目录；

3. 在 `META-INF/services` 目录下新建`org.apache.shardingsphere.sharding.spi.ShardingAlgorithm`文件，并将实现类的全路径写到文件中；

   ![自定义算法](自定义算法.png)

4. 将上述 Java 文件打包成 jar 包；

5. 将上述 jar 包拷贝至 ShardingSphere-Proxy 解压后的 `ext-lib/` 目录；

## DistSQL

> DistSQL（Distributed SQL）是 Apache ShardingSphere 特有的操作语言。 它与标准 SQL 的使用方式完全一致，用于提供增量功能的 SQL 级别操作能力;
>
> DistSQL 细分为 RDL、RQL、RAL 和 RUL 四种类型:
>
> - RDL:Resource & Rule Definition Language，负责资源和规则的创建、修改和删除。
> - RQL:Resource & Rule Query Language，负责资源和规则的查询和展现。
> - RAL:Resource & Rule Administration Language，负责强制路由、熔断、配置导入导出、数据迁移控制等管理功能。
> - RUL:Resource Utility Language，负责 SQL 解析、SQL 格式化、执行计划预览等功能。

### 说明

可通过`DistSQL`动态地调整Proxy上的资源、规则等配置，具体语法见官网。

# 易错点

> 版本：5.1.2

## 行表达式

使用`Groovy `的语法，自行搜索。

## proxy的数据库名

`config-xxx.yaml`配置文件中，`databaseName`即为创建的数据库名，在**Zookeeper**的`metadata`节点下可以看到，navicat或应用程序连接proxy时，通过这个数据库名连接。

## 配置文件加载优先级

`mode.overwrite`属性为`true`时，会使用本地配置覆盖Zookeeper中的配置，否则优先加载**Zookeeper**中的配置。

## 分片和读写分离配合使用

分片和读写分离配合使用时，分片规则处使用的数据源名称，要为读写分离规则处定义的数据源。

```
...

rules:
- !SHARDING
  tables:
    t_mt_log:
      # 这里的readwrite_ds为读写分离规则处配置的数据源
      actualDataNodes: readwrite_ds$->{0..1}.t_mt_log
- !READWRITE_SPLITTING
  dataSources:
    readwrite_ds:
      type: Static
      
...
```

## 广播表规则

**issue19753**:

广播表需存在所有的数据源中，且表结构必须一致；

若一张表存在于所有数据源中，但没有配置为广播表，则该表将被视为单表，且只使用任一数据源中的该表；

## 使用pg的问题

必须在sql中指定表所在的schema，否则可能出现不可预知的问题，例如**不能生成分布式主键**等，见`issue19541`;

```sql
# 必须指定sms
insert into sms.t_order...
```

