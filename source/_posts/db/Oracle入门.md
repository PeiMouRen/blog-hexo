---
title: Oracle入门
date: 2022-08-17 15:30:12
categories: 数据库
tags:
  -	Oracle
---

# 常用脚本

## 查看版本

```sql
select * from v$version;
```

## 查看字符集

对应环境变量`NLS_LANG`；

```sql
select userenv('language') from dual;
```

## dual

`dual`为oralce内部的一张表，为了符合语法，当查询内容不存在任何一张表中时，需要`from dual`;

```sql
select sysdate; -- error
select sysdate from dual; -- success
```

## 批量插入

方式一：

```sql
insert all 
	into t_test(name, age, remark) values('hehe111', 12, null)
	into t_test(name, age, remark) values('hehe33', 13, 123)
	into t_test(name, age, remark) values('hehe3333', 14, 223)
select 1 from dual;
```

方式二：

```sql
insert into t_test(name, age, day)
select name, age, day from 
(
	select 'aa' as name, 12 as age, '2023-03-20' as day from dual 
	union all 
	select 'bb' as name, 13 as age, '2023-03-20' as day from dual 
	union all
	select 'cc' as name, 14 as age, '2023-03-20' as day from dual 
)
```



# 常用函数

## 日期相关

|  函数   |      描述      |                          示例                           |
| :-----: | :------------: | :-----------------------------------------------------: |
| to_date | 字符串转为日期 | to_date('2022-02-22 13:20:33', 'yyyy-mm-dd hh24:mi:ss') |
| to_char | 日期转为字符串 |        to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss')        |
|         |                |                                                         |

# 服务器操作

```bash
# 不登录连接
sqlplus /nolog
# 用xxx用户登录
conn xxx/xxx
show user;
```

# 异常处理

## connection reset

**描述：**linux下连接oracle时可能会报这个错误。

**解决方法：**

- 修改`${JAVA_HOME}/lib/security/java.security`文件中`securerandom.source`的值为`file:/dev/./urandom`。（已尝试）
- 启动java时添加参数`-Djava.security.egd=file:/dev/./urandom`。（未尝试）
