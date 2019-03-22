---
title: MySQL 导出表结构和表数据 mysqldump用法
date: 2019-03-22 22:51:12
tags: MySQL
---

命令行下具体用法如下：

mysqldump -u用戶名 -p密码 -d 数据库名 表名 > 脚本名;



### 导出整个数据库结构和数据

```shell
mysqldump -h localhost -uroot -p123456 database > dump.sql
```

### 导出单个数据表结构和数据

```shell
mysqldump -h localhost -uroot -p123456  database table > dump.sql
```

### 导出整个数据库结构（不包含数据）

```shell
mysqldump -h localhost -uroot -p123456  -d database > dump.sql
```

### 导出单个数据表结构（不包含数据）

```shell
mysqldump -h localhost -uroot -p123456  -d database table > dump.sql
```

