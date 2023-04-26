---
title: Oracle迁移至PostgreSQL
date: 2022-08-16 16:07:42
categories:
  - 中间件
  - 数据迁移
tags:
  - Oracle
  - PostgreSQL
  -	Ora2Pg
  - 数据迁移
---

[toc]

# 数据库迁移

> 使用工具`Ora2Pg`进行数据库迁移；
>
> `Ora2Pg`:**Moves Oracle and MySQL database to PostgreSQL**
>
> `Ora2Pg`官网：`https://ora2pg.darold.net/`

## 搭建Ora2Pg环境

### 安装oracle client

```bash
# 安装oracle client

# oracle instant client下载地址
# https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html
# rpm包默认会安装到/usr/lib/oracle/11.2/client64目录下
rpm -ivh oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm
rpm -ivh oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm
rpm -ivh oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm
rpm -ivh oracle-instantclient11.2-jdbc-11.2.0.4.0-1.x86_64.rpm
mkdir -p /usr/lib/oracle/11.2/client64/network/admin
vi /usr/lib/oracle/11.2/client64/network/admin/tnsnames.ora # 配置监听文件，内容见下
vi /etc/profile # 配置环境变量，添加内容见下
source /etc/profile
# 测试连接oracle
sqlplus username/password@tnsnames.ora中的监听器名称
sqlplus username/password@ip:port/服务名称
```

```properties
# tnsnames.ora文件内容
监听器名称 = 
 (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = ip地址)(PORT = 端口号))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = 服务名) 
    ) 
)

# /etc/profile添加配置
export ORACLE_HOME=/usr/lib/oracle/11.2/client64
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export NLS_LANG=AMERICAN_AMERICA.UTF8
```

### 安装Perl模块

```bash
# 查看perl版本，需要5.10及以上的版本
perl -v

# 安装相关模块
yum -y install perl-YAML perl-Term-ReadLine perl-version perl-libs perl-devel perl-Test-* perl-Pod-* perl-ExtUtils-* perl-CGI perl-CPAN

# 安装DBI
# 下载地址https://github.com/perl5-dbi/dbi/tags
mkdir -p /usr/local/perl/dbi
cd /usr/local/perl/dbi
tar -zxvf DBI-1.643.tar.gz
cd DBI-1.643
perl Makefile.PL
make && make install

# 安装DBD::Oracle
# 下载地址https://github.com/singingfish/DBD-Oracle/tags
# 		https://cpan.metacpan.org/authors/id/Z/ZA/ZARQUON/DBD-Oracle-1.83.tar.gz
mkdir -p /usr/local/perl/dbdoracle
cd /usr/local/perl/dbdoracle/
tar -zxvf DBD-Oracle-1.83.tar.gz
cd DBD-Oracle-1.83
perl Makefile.PL
make && make install

# 安装DBD::Pg
# 下载地址：https://github.com/bucardo/dbdpg/tags
yum install postgresql-devel
mkdir -p /usr/local/perl/dbdpg
cd /usr/local/perl/dbdpg/
tar -zxvf dbdpg-3.15.1.tar.gz
cd dbdpg-3.15.1
perl Makefile.PL
make && make install

# 安装ora2pg
# 下载地址https://github.com/darold/ora2pg/tags
mkdir -p /usr/local/perl/ora2pg
cd /usr/local/perl/ora2pg/
tar -zxvf ora2pg-22.0.tar.gz
cd ora2pg-22.0
perl Makefile.PL
make && make install
```

### 查看已安装的perl模块

```bash
vi /tmp/check.pl # 内容见下文
perl /tmp/check.pl
```

**check.pl内容**

```shell
#!/usr/bin/perl
use strict;
use ExtUtils::Installed;
my $inst=ExtUtils::Installed->new();
my @modules = $inst->modules();
foreach(@modules){
    my $ver = $inst->version($_) || "???";
    printf("%-12s -- %s\n",$_,$ver);
}
exit;
```

### 删除安装的perl模块

```bash
# 在安装目录下执行
make uninstall|grep unlink|sh
```

## 使用Ora2Pg

### 修改配置文件

```bash
cp /etc/ora2pg/ora2pg.conf.dist /etc/ora2pg/ora2pg.conf
vi /etc/ora2pg/ora2pg.conf # 修改内容见下
```

```
ORACLE_HOME	/usr/lib/oracle/11.2/client64
ORACLE_DSN	dbi:Oracle:host=172.17.3.3;sid=orcl;port=1521
ORACLE_USER	ir # 该用户需为DBA用户
ORACLE_PWD	ir
SCHEMA		ir
EXPORT_SCHEMA	1
CREATE_SCHEMA	1
TYPE		TABLE # 导出表结构
PG_DSN		dbi:Pg:dbname=genericdb;host=192.168.5.112;port=5432
PG_USER	 	ir
PG_PWD		ir
FILE_PER_FKEYS		1 # 将外键放到单独的sql脚本中，等数据迁移完毕后，再给表增加外键
```

### 导出结构和数据

```bash
# 1、导出结构
# 设置ora2pg.conf中TYPE参数
TYPE TABLE,VIEW,SEQUENCE,TRIGGER,FUNCTION,PROCEDURE,TABLESPACE,PARTITION
# 执行后会在当前目录下生成对应的sql脚本和ora2pg.log
ora2pg -d -l ora2pg.log
# pg创建用户
create user ir with password 'ir' superuser;
# 导入结构，分别执行导出的sql脚本
# 注意检查一下PARTITION_output.sql中是否存在重名的分区表，若存在，则手动调整一下分区表名称
# 注意：pg中，建表语句中表名如果不加双引号限制，则创建的表名默认为全小写，而oracle中为全大写
# 存放外键的sql脚本要等到表数据迁移完毕后再执行
./psql -d genericdb -h 192.168.5.112 -p 5432 -U ir -f /tmp/xxx.sql

# 2、导出数据
# 设置ora2pg.conf中TYPE参数
TYPE INSERT
# 执行后数据会自动插入到pg库中
ora2pg -d -l ora2pg.log
```

## pg库方面的改动

### 修改配置文件

调整存放数据的`data`目录下的配置文件，允许其他ip连接pg

```
# 1. 修改data目录下pg_hba.conf,增加IPv4配置
host    all         all          0.0.0.0/0        trust
# 2. 修改data目录下的postgresql.conf
listen_addresses = '*'
```

### 修改编码

若`ora2pg`导出到`pg`失败，可考虑是pg库编码的问题。

---

**命令行查看编码**

```bash
psql -l
```

---

**sql查看编码**

```sql
select * from pg_database where datname = 'postgres';
```

---

**修改编码为utf8**

```sql
update pg_database set encoding = pg_char_to_encoding('UTF8'), datcollate = 'en_US.UTF-8', datctype = 'en_US.UTF-8' where datname = 'postgres';
```

# 代码调整

## 切换数据源

- mvn依赖

```xml
<!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.3.6</version>
</dependency>
```

- jar包下载

```http
https://jdbc.postgresql.org/download.html
```

- url

 `jdbc:postgresql://ip:port/dbname`

- driver class

`org.postgresql.Driver`

## sql调整

| 对比项     | oracle                 | postgresql                               |
| ---------- | ---------------------- | ---------------------------------------- |
| dual       | select 1 + 1 from dual | select 1 + 1                             |
| 系统时间   | sysdate                | now(), current_timestamp, localtimestamp |
| 时间加减   | sysdate + 1            | now() + INTERVAL '1 DAY'                 |
| 获取序列值 | sequence.nextval       | nextval('sequence')                      |
| 序列当前值 | sequence.currval       | currval('sequence')                      |
| 判断空值   | nvl(exp1, exp2)        | coalesce(exp1, exp2)                     |
| 检索字符串 | instr('str1', 'str2')  | strpos('str1', 'str2')                   |
| 分页       | rownum                 | limit pageSize offset pageSize * pageNum |
| 行号       | rownum                 | row_number() over() as rownum            |
| 条件判断   | decode()               | case when then else end                  |

## jndi配置

> 若使用jndi数据源，则直接修改tomcat的配置文件即可，注意需要在tomcat的lib包下放pg的jar包。

`${tomcathome}/conf/context.xml`添加配置：

```xml
<Context>

    <!-- ... -->
    
    <Resource name="jdbc/ir" auth="Container" 
              type="javax.sql.DataSource"
              username="ir" password="ir"
              driverClassName="org.postgresql.Driver"
              url="jdbc:postgresql://192.168.5.112:5432/genericdb" 
              maxTotal="8" maxIdle="4" />
</Context>
```

## 其他

1. `hibernate`框架需要调整`sessionFactory`中的`hibernate.dialect`属性为`org.hibernate.dialect`包下对应版本的`PostgreSQLXXDialet`。

2. pg在使用聚合函数时的问题；

   **pg要求sql语句中出现聚合函数或order by时，除非group by的字段是主键，否则select、order by等关键字后面跟的字段必须要在group by关键字中声明。**

   ```sql
   -- 例如存在如下表结构和数据
   create table t_user(
   	id serial,
       name varchar(20),
       logdate date
   );
   insert into t_user(name, logdate) values
   ('zhangsan', '20220606'),('zhangsan', '20220607'),('lisi', '20220607');
   
   -- ERROR:  column "t_user.id" must appear in the GROUP BY clause or be used in an aggregate function
   select id, name, logdate, count(logdate) from t_user group by logdate;
   
   -- success
   select logdate, count(logdate) from t_user group by logdate;
   
   -- ERROR:  column "t_user.name" must appear in the GROUP BY clause or be used in an aggregate function
   select logdate, count(logdate) from t_user group by logdate order by name;
   
   -- 现在将表t_user中的id属性设置为主键
   alter table t_user add primary key (id);
   
   -- success
   select id, name, logdate, count(logdate) from t_user group by id order by name;
   ```

3. pg在使用update set时的问题；

   **pg要求update时，不能在更新列的说明中包含表的名称（别名也不行）。**

    ``` sql
   -- ERROR:  column "t" of relation "t_service" does not exist
   update T_SERVICE t set t.SERVICENAME = '业务0002' where t.SERVICECODE = '1000038436';
   -- success
   update T_SERVICE t set SERVICENAME = '业务0002' where t.SERVICECODE = '1000038436';
    ```

4. pg在获取序列下个值时的问题；

   **pg要求调用nextval('sequence')时，不能在一个只读的事务中，即spring在配置事务时，涉及到序列的方法不能设置为只读事务。**

   **例：<tx:method name="getNext*Id" read-only="false"/>**

5. 标识符和关键字问题

   pg和oracle中**关键词和不被引号修饰的标识符**都是大小写不敏感的，但pg会统一转换为小写，而oracle会统一转换为大写，所以要特别注意sql中用引号修饰的标识符。

   ```sql
   # oracle
   create table T_TEST(id int, name varchar2(20)); # 创建T_TEST表
   select * from t_test; # 成功，t_test会自动转为T_TEST
   select * from "t_test"; # 失败，表t_test不存在
   # pg
   create table T_TEST(id serial, name varchar(20)); # 创建t_test表
   select * from t_test; # 成功
   select * from "T_TEST"; # 失败，表T_TEST不存在
   ```

   

​	

