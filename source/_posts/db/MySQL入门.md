---
title: MySQL入门
date: 2022-08-16 15:26:55
categories: 
  - 数据库
tags:
  - MySQL
---

[toc]





# 功能记录

## 查询表分区

```sql
SELECT * FROM information_schema.`PARTITIONS` WHERE table_schema = SCHEMA() AND table_name = 'employees';
```

## 查看区分大小写情况

```sql
-- lower_case_table_names为1表示不区分大小写
SHOW VARIABLES LIKE '%case%';
```

