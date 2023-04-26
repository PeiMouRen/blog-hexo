---
title: MySQL迁移至PostgreSQL
date: 2022-08-16 15:22:55
categories:
  - 中间件
  - 数据迁移
tags:
  - 数据迁移
  - MySQL
  - PostgreSQL
  - pgloader
---

[toc]

# 数据迁移

## 使用pgloader工具

> 官方地址：`https://github.com/dimitri/pgloader`
>
> 文档地址：`https://pgloader.readthedocs.io/en/latest/index.html`

---

**安装**

github上有源码安装方式，本次使用docker安装。

```bash
docker pull dimitri/pgloader # 拉取镜像
docker run -it --name mypgloader dimitri/pgloader /bin/bash # 启动并进入容器
exit # 退出容器
```

---

**使用**

```bash
# 进入到容器内
docker exec -it mypgloader /bin/bash
# 查看pgloader版本和帮助信息
pgloader -V
pgloader -h
# 编辑配置文件，内容见下
vim /tmp/pgloader.load
```

**pgloader.load**

> 配置文件说明见说明文档：`https://pgloader.readthedocs.io/en/latest/ref/mysql.html#`

```
LOAD DATABASE
     FROM mysql://isms:POSTisms2019@172.19.4.37:3306/isms
     INTO postgresql://isms:POSTisms2019@192.168.1.157:5432/postgres

WITH include drop, create tables, create indexes, foreign keys,
     uniquify index names, downcase identifiers, workers = 8, concurrency = 1
;
```

**pg库准备**

```sql
# 创建isms用户、模式
CREATE USER isms WITH PASSWORD 'POSTisms2019' SUPERUSER;
CREATE SCHEMA isms;
ALTER SCHEMA isms OWNER TO isms;
```

**开始迁移数据**

```bash
# 容器内部执行
pgloader /tmp/pgloader.load
```

## pgloader存在的问题及解决方法

### 问题

1. 只能迁移表结构和表数据，视图、函数等无法迁移；
2. mysql中的分区表迁移后没有分区；

### 解决方法

1. 迁移视图问题，可通过数据库连接工具（如navicat）批量导出视图的ddl语句，再针对pg与mysql语法的不同之处进行适当修改，最后在pg库中手动执行ddl语句来创建视图；

   - 删除`CREATE`和`VIEW`关键字之间的内容；
   - 删除所有的反引号`` ` ``;
   - `FROM`后尽量不要跟括号`()`，否则有可能报错；
   - `JOIN`后要跟`ON`；
   - 替换用到的函数，如格式化日期函数等；

   ```sql
   -- mysql create view
   CREATE ALGORITHM = UNDEFINED DEFINER = `isms` @`%` SQL SECURITY DEFINER VIEW `v_pro_cpsrv` AS SELECT
   `t_cp_service`.`CP_SERVICE_CODE` AS `CPSRVCODE`,
   `t_cp`.`PROVINCE_CODE` AS `PROVINCE_CODE` 
   FROM
   	( `t_cp_service` JOIN `t_cp` ) 
   WHERE
   	( `t_cp_service`.`CPCORP_ID` = `t_cp`.`CPCORP_ID` );
   
   
   -- pg create view
   CREATE VIEW v_pro_cpsrv AS SELECT
   t_cp_service.CP_SERVICE_CODE AS CPSRVCODE,
   t_cp.PROVINCE_CODE AS PROVINCE_CODE 
   FROM
   	 t_cp_service JOIN t_cp  
   ON
   	 (t_cp_service.CPCORP_ID = t_cp.CPCORP_ID);
   ```

2. 分区表问题，可先查出mysql中的分区表，再将pg中对应表删除后重新创建；

   1. 查询mysql中的分区表；

      ```sql
      SELECT distinct table_name FROM information_schema.PARTITIONS where table_schema = 'isms' and partition_method is not null;
      ```

   2. 使用navicat导出pg中对应表的DDL；

      ```sql
      CREATE TABLE "isms"."t_hy_order" (
        "order_id" numeric(10,0) NOT NULL,
        "service_code" varchar(32) COLLATE "pg_catalog"."default",
        "cp_service_code" varchar(32) COLLATE "pg_catalog"."default",
        "op_service_code" varchar(32) COLLATE "pg_catalog"."default",
        "phone" varchar(32) COLLATE "pg_catalog"."default",
        "phone_type" numeric(10,0),
        "action" numeric(10,0),
        "access_mode" numeric(10,0),
        "opcorp_id" varchar(6) COLLATE "pg_catalog"."default",
        "order_flag" numeric(10,0),
        "order_time" timestamptz(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
        "attach_info_type" numeric(10,0),
        "attach_info" varchar(64) COLLATE "pg_catalog"."default",
        "remark" varchar(64) COLLATE "pg_catalog"."default",
        "branch_no" varchar(16) COLLATE "pg_catalog"."default",
        "oper_no" varchar(16) COLLATE "pg_catalog"."default",
        "fee_value" numeric(6,0),
        "province_code" char(6) COLLATE "pg_catalog"."default",
        "city_code" char(6) COLLATE "pg_catalog"."default",
        "town_code" varchar(9) COLLATE "pg_catalog"."default",
        CONSTRAINT "idx_37895_primary" PRIMARY KEY ("order_id", "order_time")
      )
      ;
      
      ALTER TABLE "isms"."t_hy_order" 
        OWNER TO "isms";
      
      CREATE INDEX "idx_37895_attach_info" ON "isms"."t_hy_order" USING btree (
        "attach_info" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
      );
      
      CREATE INDEX "idx_37895_phone" ON "isms"."t_hy_order" USING btree (
        "phone" COLLATE "pg_catalog"."default" "pg_catalog"."text_ops" ASC NULLS LAST
      );
      ```

   3. 删除pg中未进行分区的表；

      ```sql
      DROP TABLE t_hy_order;
      ```

   4. 结合mysql中表的分区信息，对第2步导出的DDL信息进行改造；

      ```sql
      CREATE TABLE "t_hy_order" (
        "order_id" numeric(10,0) NOT NULL,
        "service_code" varchar(32),
        "cp_service_code" varchar(32),
        "op_service_code" varchar(32),
        "phone" varchar(32),
        "phone_type" numeric(10,0),
        "action" numeric(10,0),
        "access_mode" numeric(10,0),
        "opcorp_id" varchar(6),
        "order_flag" numeric(10,0),
        "order_time" timestamptz(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
        "attach_info_type" numeric(10,0),
        "attach_info" varchar(64),
        "remark" varchar(64),
        "branch_no" varchar(16),
        "oper_no" varchar(16),
        "fee_value" numeric(6,0),
        "province_code" char(6),
        "city_code" char(6),
        "town_code" varchar(9),
        CONSTRAINT "idx_37895_primary" PRIMARY KEY ("order_id", "order_time")
      ) partition by range(ORDER_TIME);
      
      create table t_hy_order_P201901 partition of t_hy_order for values from (MINVALUE) to ('2019-02-01');
      create table t_hy_order_P201902 partition of t_hy_order for values from ('2019-02-01') to ('2019-03-01');
      
      CREATE INDEX "idx_37895_attach_info" ON "t_hy_order" ("attach_info");
      CREATE INDEX "idx_37895_phone" ON "t_hy_order" ("phone");
      ```

   5. 重新在pg中创建分区表；

   6. 使用pgloader重新导入分区表的数据；

      **pgloader.load修改如下**

      > WITH后参数改变；
      >
      > INCLUDING ONLY TABLE NAMES MATCHING后跟需要重新导数的表名；

      ```
      LOAD DATABASE
           FROM mysql://isms:POSTisms2019@172.19.4.37:3306/isms
           INTO postgresql://isms:POSTisms2019@192.168.1.157:5432/postgres
      
      WITH include no drop, create no tables, create no indexes, no foreign keys,
           downcase identifiers, workers = 8, concurrency = 1
      INCLUDING ONLY TABLE NAMES MATCHING 't_hy_order', 't_hy_order_his', 't_mo_log', 't_mt_log', 't_mt_log_detail', 't_op_order', 't_op_whitelist', 't_report_log', 't_tj_cpsend_minute', 't_tj_opsend_minute', 't_tj_report_minute' 
      ;
      ```

# 项目调整

## 替换maven坐标

```xml
<!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.3.6</version>
</dependency>
```

## 调整数据源

- url

 `jdbc:postgresql://ip:port/dbname`

- driver class

`org.postgresql.Driver`

## Hibernate调整方言

```xml
<bean id="sessionFactory" class="..."> 
    ...
    <prop key="hibernate.dialect">
        org.hibernate.dialect.PostgreSQL10Dialect</prop>
    ...
        
</bean>
```



# sql改动

> 由于本次调整的代码是基于Hibernate框架的，没有涉及到太多直接写sql的地方，所以下面关于sql的改动点可能不全，后续发现后及时补充。

## 函数、关键字替换

|   mysql   |        pg        |    说明    |
| :-------: | :--------------: | :--------: |
|    &&     |       and        |   逻辑与   |
|   \|\|    |        or        |   逻辑或   |
|  concat   |       \|\|       | 字符串连接 |
| limit 5,0 | limit 5 offset 0 |    分页    |

## 其他改动点

1. mysql使用单引号`'`或双引号`"`来引用属性值（例如`where name = "Tom"`，或`where name = 'Tom'`）；pg只能使用单引号`'`来引用数值（即`where name = 'Tom'`），pg中的双引号`"`用于修饰表名、列名（如`where "name" = 'Tom'`），来保留修饰字段的大小写，否则会统一转为小写；

   ```sql
   -- mysql
   select * from user where name = "Tom"; -- success
   select * from user where name = 'Tom'; -- success
   
   -- pg
   select * from user where name = "Tom"; -- ERROR:  column "Tom" does not exist
   select * from user where name = 'Tom'; -- success
   ```

2. mysql中使用反引号`` ` ``来引用数据库名、表名、字段名，pg中则不允许；

   ```sql
   -- mysql
   SELECT * FROM `isms`.`isms_order` WHERE `title` = "神州行"; -- success
   
   -- pg
   select * from `t_province` where province_name = '北京'; -- ERROR:  syntax error at or near "`"
   ```

3. mysql`lower_case_table_names`为`1`时，大小写不敏感（包括关键字、数据库名、表名、列名、列值等），pg除列值外大小写不敏感；

   ```sql
   -- insert into user (name, age) values ('Tom', 22);
   
   -- mysql
   select * from user where name = 'Tom'; -- 成功查询到记录
   select * from user where name = 'tom'; -- 成功查询到记录
   
   -- pg
   select * from user where name = 'Tom'; -- 成功查询到记录
   select * from user where name = 'tom'; -- 查不到记录
   ```

   
