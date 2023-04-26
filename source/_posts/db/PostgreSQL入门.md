---
title: PostgreSQL入门
date: 2022-08-16 16:09:56
categories:
  - 数据库
tags:
  - PostgreSQL
---

# PostgreSQL部署

## 下载

```http
# 官方下载
https://www.postgresql.org/download/
# 中文社区下载
http://www.postgres.cn/v2/download
```

## 安装

> 使用操作系统用户postgres创建并初始化数据库时，pg会默认创建一个postgres用户，但该数据库用户没有设置密码，所以需要手动创建数据库用户或者给数据库postgres用户设置密码。

```bash
# 下载并解压
wget https://ftp.postgresql.org/pub/source/v12.2/postgresql-12.2.tar.bz2
tar xjvf postgresql*.bz2
cd postgresql-12.2
# 编译安装
./configure --prefix=/usr/local/pgsql # /usr/local/pgsql为安装目录
make
make install
# 配置数据库用户及目录
adduser postgres
mkdir /usr/local/pgsql/data
chown -R postgres:postgres /usr/local/pgsql/data
# 使用postgres用户初始化并启动数据库

su postgres
# 设置环境变量
#PAHT = /usr/local/pgsql/bin:$PATH
#export PATH
#PGDATA = /usr/local/pgsql/data
#export PGDATA
# -D选项缺省时，使用PGDATA环境变量
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
# 创建并进入数据库
/usr/local/pgsql/bin/createdb testdb
/usr/local/pgsql/bin/psql testdb

```

# 配置文件

`initdb`执行完成后，会在`PGDATA`目录下生成相应的配置文件，常见需要调整的配置文件有`pg_hba.conf`和`postgresql.conf`两个。

****

**pg_hba.conf**

```
# 允许所有ip连接
# IPv4 local connections:
host    all             all             0.0.0.0/0               md5
```

---

**postgresql.conf**

```
# 监听所有TCP/IP连接
listen_addresses = '*'
port=5432
# 开启日志记录
logging_collector = on
log_directory = 'pg_log'
```



# PostgreSQL常用命令

```bash
# 启动/停止/重启数据库
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data start|stop|restart
# 创建数据库
/usr/local/pgsql/bin/createdb testdb
# 查看数据库列表
/usr/local/pgsql/bin/psql -l
# 连接数据库
/usr/local/pgsql/bin/psql testdb
```

# PostgreSQL数据类型

## 数字类型

|        名字        | 大小  |        描述        |                     范围                     |
| :----------------: | :---: | :----------------: | :------------------------------------------: |
|     `smallint`     | 2字节 |     小范围整数     |               -32768 to +32767               |
|     `integer`      | 4字节 |   整数的典型选择   |          -2147483648 to +2147483647          |
|      `bigint`      | 8字节 |     大范围整数     | -9223372036854775808 to +9223372036854775807 |
|     `decimal`      | 可变  | 用户指定精度，精确 |  最高小数点前131072位，以及小数点后16383位   |
|     `numeric`      | 可变  | 用户指定精度，精确 |  最高小数点前131072位，以及小数点后16383位   |
|       `real`       | 4字节 |  可变精度，不精确  |                6位十进制精度                 |
| `double precision` | 8字节 |  可变精度，不精确  |                15位十进制精度                |
|   `smallserial`    | 2字节 |  自动增加的小整数  |                   1到32767                   |
|      `serial`      | 4字节 |   自动增加的整数   |                1到2147483647                 |

## 序数类型

`smallserial`、`serial`和`bigserial`类型不是真正的类型，它们只是为了创建唯一标识符列而存在的方便符号（类似其它一些数据库中支持的`AUTO_INCREMENT`属性）。 在目前的实现中，下面一个语句：

```sql
CREATE TABLE tablename (
    colname SERIAL
);
```

等价于以下语句：

```sql
CREATE SEQUENCE tablename_colname_seq AS integer;
CREATE TABLE tablename (
    colname integer NOT NULL DEFAULT nextval('tablename_colname_seq')
);
ALTER SEQUENCE tablename_colname_seq OWNED BY tablename.colname;
```

因此，我们就创建了一个整数列并且把它的缺省值安排为从一个序列发生器取值。应用了一个`NOT NULL`约束以确保空值不会被插入（在大多数情况下你可能还希望附加一个`UNIQUE`或者`PRIMARY KEY`约束避免意外地插入重复的值，但这个不是自动发生的）。最后，该序列被标记为“属于”该列，这样当列或表被删除时该序列也会被删除。

## 字符类型

|                 名字                 |      描述      |
| :----------------------------------: | :------------: |
| `character varying(n)`, `varchar(n)` |  有限制的变长  |
|      `character(n)`, `char(n)`       | 定长，空格填充 |
|                `text`                |    无限变长    |

## 日期/时间类型

|   名字    |           描述           |          示例           |
| :-------: | :----------------------: | :---------------------: |
| timestamp | 包括日期和时间（无时区） | 1999-01-08 04:05:06.332 |
|   date    |           日期           |  1999-01-08、19990108   |
|   time    |       一天中的时间       |      04:05:06.789       |
| interval  |         时间间隔         |    interval '1 day'     |

### 间隔单位缩写

| 缩写 |         含义         |
| :--: | :------------------: |
|  Y   |          年          |
|  M   |  月（在日期部分中）  |
|  W   |          周          |
|  D   |          日          |
|  H   |         小时         |
|  M   | 分钟 (在时间部分中） |
|  S   |          秒          |

## 布尔类型

|  名称   |                             描述                             |
| :-----: | :----------------------------------------------------------: |
| boolean | 输入：<br />真：true、yes、on、1或者这些字符的唯一前缀，不区分大小写<br />假：false、no、off、0或者这些字符的唯一前缀，不区分大小写<br />输出：<br />真：t<br />假：f<br /> |

## 常用函数和操作符

### 字符串相关

|                             名称                             |                             描述                             |                      案例                       |                结果                 |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :---------------------------------------------: | :---------------------------------: |
|                             \|\|                             |                             串接                             |              'Post' \|\| 'greSQL'               |             PostgreSQL              |
|                    bit_length(*`string`*)                    |                          串中的位数                          |              `bit_length('jose')`               |                `32`                 |
|   `char_length(`string`)` or `character_length(`string`)`    |                          串中字符数                          |              `char_length('jose')`              |                 `4`                 |
|                      length(*`string`*)                      |                     *`string`*中的字符数                     |                `length('jose')`                 |                 `4`                 |
|                      lower(*`string`*)                       |                    将字符串转换为小写形式                    |                 `lower('TOM')`                  |                `tom`                |
|                      upper(*`string`*)                       |                    将字符串转换成大写形式                    |                 `upper('tom')`                  |                `TOM`                |
| overlay(*`string`* placing *`string`* from `int` [for `int`]) |                           替换子串                           | `overlay('Txxxxas' placing 'hom' from 2 for 4)` |              `Thomas`               |
|            position(*`substring`* in *`string`*)             |                         定位指定子串                         |          `position('om' in 'Thomas')`           |                 `3`                 |
|        substring(*`string`* [from `int`] [for `int`])        |                           提取子串                           |       `substring('Thomas' from 2 for 3)`        |                `hom`                |
|          substr(*`string`*, *`from`* [, *`count`*])          |                           提取子串                           |           `substr('alphabet', 3, 2)`            |                `ph`                 |
| trim([leading \|trailing\|both] [*`characters`*] from *`string`*) | 从*`string`*的开头、结尾或者两端（`both`是默认值）移除只包含*`characters`*（默认是一个空格）中字符的最长字符串 |        `trim(both 'xyz' from 'yxTomxx')`        |                `Tom`                |
|                      ascii(*`string`*)                       | 参数第一个字符的ASCII代码。对于UTF8返回该字符的Unicode代码点。对于其他多字节编码，该参数必须是一个ASCII字符。 |                  `ascii('x')`                   |                `120`                |
|                          chr(`int`)                          | 给定代码的字符。对于UTF8该参数被视作一个Unicode代码点。对于其他多字节编码该参数必须指定一个ASCII字符。NULL (0) 字符不被允许，因为文本数据类型不能存储这种字节。 |                    `chr(65)`                    |                 `A`                 |
|              left(*`str`* `text`, *`n`* `int`)               |  返回字符串中的前*`n`*个字符。当*`n`*为负时，将返回除了最后  |          *`n`*\|个字符之外的所有字符。          |         `left('abcde', 2)`          |
|              right(*`str`* `text`, *`n`* `int`)              |  回字符串中的最后*`n`*个字符。如果*`n`*为负，返回除最前面的  |           *`n`*\|个字符外的所有字符。           |         `right('abcde', 2)`         |
|                       md5(*`string`*)                        |        计算*`string`*的 MD5 哈希，返回十六进制的结果         |                  `md5('abc')`                   | `900150983cd24fb0 d6963f7d28e17f72` |
|         repeat(*`string`* `text`, *`number`* `int`)          |               重复*`string`*指定的*`number`*次               |                `repeat('Pg', 4)`                |             `PgPgPgPg`              |
|  replace(*`string`* `text`, *`from`* `text`, *`to`* `text`)  |     将*`string`*中出现的所有子串*`from`*替换为子串*`to`*     |      `replace('abcdefabcdef', 'cd', 'XX')`      |           `abXXefabXXef`            |
|                       reverse(*`str`*)                       |                      返回反转的字符串。                      |               `reverse('abcde')`                |               `edcba`               |
|             starts_with(*`string`*, *`prefix`*)              |           如果*`string`*以*`prefix`*开始则返回真。           |        `starts_with('alphabet', 'alph')`        |                 `t`                 |
|             to_hex(*`number`* `int` or `bigint`)             |            将*`number`*转换到它等效的十六进制表示            |              `to_hex(2147483647)`               |             `7fffffff`              |
|                                                              |                                                              |                                                 |                                     |
|                                                              |                                                              |                                                 |                                     |

### 日期/时间相关

|          名称           |             描述             |                                     |
| :---------------------: | :--------------------------: | ----------------------------------- |
| to_date(`text`, `text`) |       把字符串转成日期       | to_date('2022-01-01', 'YYYY-MM-DD') |
| to_char(`date`, `text`) |       把日期转成字符串       | to_char(date1, 'YYYYMMDD')          |
|      current_date       |           当前日期           |                                     |
|      current_time       |    当前时间with time zone    |                                     |
|    current_timestamp    | 当前日期和时间with time zone |                                     |
|        localtime        |           当前时间           |                                     |
|     localtimestamp      |        当前日期和时间        |                                     |
|          now()          | 当前日期和时间with time zone |                                     |
|                         |                              |                                     |



# PostgreSQL常用sql脚本

## 查看数据库版本

```sql
select * from version();
```

## 查看所有表信息

```sql
select * from pg_tables;
```

## 创建表

```sql
create table test(
    id int primary key, -- primary key表示主键
	key1 numeric default 0.0, -- default指定默认值
    key2 text default 'test' not null, -- not null表示非空约束
    -- generated always as () stored 表示生成列的生成规则
    key3 numeric generated always as (key1 / 10) stored,
    -- 列约束，constraint指定约束名称 check 表示检查约束
    key4 numeric constraint test_key4 check (key4 > 0), 
    key5 integer unique, -- unique表示唯一约束，可写为 unique (key5)
     -- 外键约束,on delete restrict表示其他表中相关数据删除后，本表数据不删除
    key6 integer references test2(id) on delete restrict,
    -- on delete cascade表示其他表中数据删除后，本表数据也要删除
    key7 integer references test3(id) on delete cascade,
    -- 表级检查约束
    constraint test_key4_key1 check (key4 > key1 and key1 > 0),
    -- 表级唯一约束
    constraint test_key4_key5 unique (key4, key5),
    -- 表级外键约束
    constraint test_key1_key5 foreign key (key1, key5) references test3(key1, key2)
);
```

## 给表、列添加注释

```sql
comment on table test is 'testtest';
comment on column test.key2 is 'key2test';
```

## 索引

```sql
CREATE INDEX test2_mm_idx ON test2 (major, minor);
DROP INDEX test2_mm_idx;
```

## 修改表

```sql
# 增加列，创建表时的一些约束都可以用在这儿
alter table test add column key8 text, add column key9 text;
# 移除列,cascade表示授权删除任何依赖该列的数据（如外键）
alter table test drop column key8 cascade;
# 增加约束
alter table test add check(key7 > 0);
alter table test alter column key7 set not null;
# 移除约束
alter table test drop constraint constraint_name;
alter table test alter column key7 drop not null; -- 移除test表key7列上的非空约束
# 修改列的默认值
alter table test alter column key1 set default 2.0;
alter table test alter column key1 drop default; -- 移除默认值
# 修改列的数据类型
alter table test alter column key1 type numeric(10, 2);
# 重命名列
alter table test rename column id to key0;
# 重命名表
alter table test rename to test1;
```

## 权限

```sql
# 重新分配test表的所有者，超级用户总是可以做到这点，普通角色只有同时是对象的当前所有者（或者是拥有角色的一个成员）以及新拥有角色的一个成员时才能做同样的事
alter table test owner to new_owner;
# 授予joe用户更新test表的权限
grant update on test to joe;
## 授权其他写法
GRANT SELECT ON mytable TO PUBLIC;
GRANT SELECT, UPDATE, INSERT ON mytable TO admin;
GRANT SELECT (col1), UPDATE (col1) ON mytable TO miriam_rw;
# 撤销public用户对test表的所有权限
revoke all on test from public;
```

## 分区

```sql
# 创建分区表，range表示范围分区，其他的还有list和hash分区
create table test2(
	id integer,
	logtime date
) partition by range(logtime);
# 创建分区
create table test2_202001 partition of test2 for values from (MINVALUE) to ('2020-01-01');
create table test2_202002 partition of test2 for values from ('2020-01-01') to ('2020-02-01');
# 删除分区
drop table test2_202001;

# 列表分区
create table test3(
	id integer,
	logtime date
) partition by range(id);
create table test3_p1 partition of test3 for values in(1,2);

# 哈希分区 MODULUS-模	REMAINDER-余数
create table test4(
	id integer,
	logtime date
) partition by hash(id);
create table test4_p1 partition of test4 for values with(MODULUS 4, REMAINDER 0);
create table test4_p2 partition of test4 for values with(MODULUS 4, REMAINDER 1);
create table test4_p3 partition of test4 for values with(MODULUS 4, REMAINDER 2);
create table test4_p4 partition of test4 for values with(MODULUS 4, REMAINDER 3);

# 创建默认分区
CREATE TABLE cities_partdef PARTITION OF cities DEFAULT
```

## 查询

```sql
# 分页
select * from account limit 3 offset 0;
# with的简单使用
with t as (
	select * from account where name = 'lisi'
)
select * from t;
```

## 事务操作

```sql
begin; -- 开启事务
update account set balance = balance - 10 where name = 'zhangsan';
savepoint my_savepoint; -- 添加保存点
update account set balance = balance + 10 where name = 'lisi';
rollback to my_savepoint; -- 回滚至保存点
update account set balance = balance + 10 where name = 'wangwu';
commit; -- 提交事务
-- rollback; 回滚事务
```

## 窗口函数

```sql
# 窗口函数与聚合函数类似，只不过窗口函数不会将结果聚合在一行输出
# 关键字有 over、rank、window、partition等
select sum(salary) over w, avg(salary) over w, rank() over w
from empsalary 
window w as (partition by depname order by salary desc);
```

## 表继承

```sql
# 一张表继承另一张表的所有列
# 关键字:inherits
# only表示只查询cities，不涉及继承于cities的其他表
create table capitals (
state int
) inherits (cities);
select * from only cities;
```

## 索引

```sql
# 创建索引
create index test3_idx_id on test3(id, name);
# 删除索引
drop index test3_idx_id;
# 唯一索引,pgsql会自动在唯一列上创建一个唯一索引
create unique index idx_name on table_name(column1, column2);
# 表达式索引
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));
CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
# 部分索引
# 使用部分索引的一个主要原因是避免索引公值。由于搜索一个公值的查询（一个在所有表行中占比超过一定百分比的值）不会使用索引，所以完全没有理由将这些行保留在索引中。这可以减小索引的尺寸，同时也将加速使用索引的查询。它也将加速很多表更新操作，因为这种索引并不需要在所有情况下都被更新
create index indx_name on tablename(columnname)
where columnname > 20;
# 覆盖索引
# PostgreSQL中的所有索引是二级索引,这意味着每个索引都是与表的主数据区（在PostgreSQL术语称为表的堆中）分开存储。这意味着在普通索引扫描中，每行检索都需要从索引和堆中取数据。 此外，虽然匹配给定的可索引WHERE条件的索引条目通常在一起靠近存储，但它们引用的表行可能在堆中的任何地方。 因此索引扫描的堆访问部分涉及到对堆的大量随机访问，这可能很慢，特别是在传统旋转媒介上。如 第 11.5 节 中所述，位图扫描尝试通过按排序的顺序进行堆访问来减少成本，但这远远不够）。
# 为了解决这种性能问题，PostgreSQL支持只用索引的扫描，这类扫描可以仅用一个索引来回答查询而不产生任何堆访问。其基本思想是直接从每一个索引项中直接返回值，而不是去参考相关的堆项
# 例如 表的x、y列上有索引
SELECT x, y FROM tab WHERE x = 'key' AND y < 42; -- 只使用索引的扫描
SELECT x, z FROM tab WHERE x = 'key'; -- 产生堆访问
# 查看执行计划
explain select * from test3 where id = 2;
```

# 角色/用户

**角色可用属性**

- login：登录权限。`create role test login;`
- superuser: 超级管理员权限.`create role test superuser;`
- createdb: 创建数据库的权限。`create role test createdb;`
- replication: 流复制的权限，一个拥有流复制权限的角色也需要被赋予登录的权限。`create role test replication login;`
- password: 设置角色登录密码。`create role test password '123456'`

```sql
# 创建角色
create role test password 'testpwd' login;
# 修改角色
create role test superuser;
# 删除角色
drop role test;
```

# 权限

## schema

pg14版本及以前的版本，所有用户对`public`schema都有`CREATE`和`USAGE`的权限，pg15以后取消了`CREATE`的权限，需要手动授权。

```sql
# 第一个public表示public schema，第二个public表示所有用户
grant CREATE ON SHCEMA public TO PUBLIC;
```

**查询/设置搜索路径**

```sql
show search_path;
SET search_path TO myschema,public;
```

# 使用pgAdmin

## 下载

```http
# windows客户端下载
https://www.pgadmin.org/download/pgadmin-4-windows/
```

## 使用

### 汉化

File -> preference -> Miscellaneous -> User language

![pgAdmin汉化](pgAdmin汉化.jpg) 

### 连接数据库

- 改动数据库配置

```bash
# 1. 修改data目录下pg_hba.conf,增加IPv4配置
host    all         all          192.168.100.117/32        trust
# 2. 修改data目录下的postgresql.conf
listen_addresses = '*'
```

- 重启数据库

```bash
/usr/local/pgsql/bin/pg_ctl restart -D /usr/local/pgsql/data
```

- pgAdmin创建连接

![pgAdmin注册服务器](pgAdmin注册服务器.jpg)

# jdbc连接pg

## jar包下载

```http
# 下载地址
https://jdbc.postgresql.org/download.html
```

## maven依赖

```xml
<!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.3.6</version>
</dependency>
```

## jdbc连接查询

**url**

`jdbc:postgresql://ip:port/dbname?stringtype=unspecified`

- `stringtype=unspecified`：不指定字符串的类型，而由服务器来推断字符串的类型，默认值为`varchar`，可用于解决sql参数不能传字符串的问题；

```java
package com.lhx.pg;


import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class PgTest {

    public static void main(String[] args) throws Exception {

        // 注册驱动
        Class.forName("org.postgresql.Driver");

        // url: jdbc:postgresql://ip:port/dbname?stringtype=unspecified
        String url = "jdbc:postgresql://192.168.5.112:5432/genericdb";
        String user = "postgres";
        String password = "postgres";
        String sql = "select * from account";
        try (
                // 获取连接
                Connection connection = DriverManager.getConnection(url, user, password);
                // 获取数据库操作对象
                PreparedStatement preparedStatement = connection.prepareStatement(sql);
                // 执行sql
                ResultSet resultSet = preparedStatement.executeQuery()
            ) {

            // 获取执行结果
            while (resultSet.next()) {
                String name = resultSet.getString("name");
                Integer balance = resultSet.getInt("balance");
                System.out.println("name: " + name + ", balance: " + balance);
            }
        }
    }
}
```

# pg搭建主从复制

> 版本：12.6
>
> 主机：172.19.3.63
>
> 从机：172.19.3.60

## 主机配置

**创建postgres用户和用户组**

```bash
# 创建用户和组，有的安装方式在安装完时会自动创建
groupadd postgres
useradd -g postgres postgres
# 切换用户
su postgres
```

**初始化并启动数据库**

```bash
./initdb -D /home/postgres/data
./pg_ctl -D /home/postgres/data -l logfile start
# 连接postgres数据库
./psql
```

**创建用于复制的用户**

```sql
-- 主服务器上创建复制用户replicator/itc123，拥有登录和复制权限
CREATE ROLE replicator LOGIN REPLICATION PASSWORD 'itc123';
```

**修改配置文件**

`/home/postgres/data/pg_hba.conf`添加复制用户

```
# Allow replication connections from localhost, by a user with the
# replication privilege.
host 	replication		replicator		172.19.3.60/32			md5
```

`/home/postgres/data/postgresql.conf`修改配置

```
listen_addresses = '*'
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /home/postgres/data/pgarchive/%f && cp %p /home/postgres/data/pgarchive/%f'
max_wal_senders = 2
wal_keep_segments = 64
max_connections = 100
```

**重启主库**

```bash
./pg_ctl -D /home/postgres/data -l logfile restart
```

## 从机配置

> 从机不用初始化，需要从主机备份数据；若已初始化，删除`/home/postgres/data`
>
> 从机是只读的

****

**创建postgres用户和用户组**

```bash
# 创建用户和组，有的安装方式在安装完时会自动创建
groupadd postgres
useradd -g postgres postgres
# 切换用户
su postgres
```

**从主库备份数据**

```bash
./pg_basebackup -h 172.19.3.63 -p 5432 -U replicator -W -F p -R -P -X stream -D /home/postgres/data
```

**修改配置文件**

`/home/postgres/data/standby.signal`

```
standby_mode = 'on'
```

`/home/postgres/data/postgresql.conf`

```
primary_conninfo = 'host=172.19.3.63 port=5432 user=replicator password=itc123'
recovery_target_timeline = 'latest'
hot_standby = on
max_standby_streaming_delay = 30s
wal_receiver_status_interval = 10s
hot_standby_feedback = on
```

**启动数据库**

```bash
./pg_ctl -D /home/postgres/data -l logfile start
```

## 验证主从

**主从查看复制状态**

```sql
-- 主库查看发送状态
select * from pg_stat_replication;
-- 从库查看接收状态
select * from pg_stat_wal_receiver;
```

```bash
## 主从库查看复制状态
./pg_controldata -D /home/postgres/data/
```

**主库操作**

```sql
create table t_test(id serial, name varchar(200));
insert into t_test(name) values('test1');
select * from t_test;
```

**从库操作**

```sql
select * from t_test;
insert into t_test(name) values('test2'); -- error,从库只读
```

## 主从切换

**停止主库**

```bash
# 模拟故障，停止主库
./pg_ctl -D /home/postgres/data -l logfile stop 
```

**升级从库为主库**

```bash
./pg_ctl promote -D /home/postgres/data
```

# pg异常处理

## 启动异常

**异常信息：**`could not open shared memory segment "/PostgreSQL.1219399659": Permission denied`

**解决方法：**修改`postgresql.conf`中`dynamic_shared_memory_type = sysv`

