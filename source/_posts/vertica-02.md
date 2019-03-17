---
title: Vertica 基本操作语句
date: 2019-03-14 10:51:43
tags: Vertica
---

# 基本介绍

基于列存储的数据库，相对于传统的基于行的数据库，它更适合在数据仓库存储方面发挥特长。基于列存储的数据库的优点：
- 对于聚集操作，比如求sum，明显基于列存储的要比基于行存储的快；
- 对于update操作，不须接触其他列值；
- 基于行存储的数据库在查询每行记录的多个列值更高效的条件是，row-size比较小，这样一次磁盘读取就可以获取整行；
- 基于行存储的数据库在insert一行的时候相对更高效，毕竟可一次写入一个连续空间，即一次single disk seek。

从实际情况上来看，基于行存储的数据库更适合OLTP（联机事务处理系统），基于列存储的数据库更适合OLAP（联机分析处理系统），比如数据仓库。除此之外，同一列必定是同一类型大小，基于列存储的数据库更容易使用高效的存储方式，与之相对，基于行存储的数据库则只能采用随机方式处理列值了。



Vertica数据库的设计特点是：
- 它是基于列的存储结构，提高了连续的record处理的性能，但是在一般事务中增加了对单独record进行update和delete的开销；
- “单独”更新（out-of-place updates）和混合存储结构，提高了查询、插入的性能，但增加了update和delete的开销；
- 压缩，减少存储开销和IO带宽开销；
- 完全无共享架构，降低对共享资源的系统竞争。

Vertica数据库运行在基于Linux的网格服务器上，目前应用于Amazon Elastic Compute Cloud的数据库管理系统。

# 基本操作

##  进入 vsql 环境

```shell
vsql -U dbadmin -w 123456
```

##  查看帮助

```shell
dbadmin=> \h
```
## 切换vertica用户，用于创建database

```shell
# 切换用户
su - dbadmin

# 进入vertica管理工具
/opt/vertica/bin/admintools
```

[创建数据库](<https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/AdministratorsGuide/AdminTools/CreatingADatabase.htm>)



# 用户与schema查询

##  查询用户

```sql
select * from v_catalog.users;
```

##  查询schema

```sql
select * from schemata;
```

##  某个schema必须附属于某个用户（user），查询用户和schema信息

```sql
SELECT u.user_name, s.schema_name
FROM users u LEFT OUTER JOIN schemata s ON u.user_name = s.schema_owner;
```



# 创建用户和schema

##  创建一个用户

```sql
# username:dev_test password:test
create user dev_test identified by 'test';
```

##  基于某个用户创建schema

```sql
create schema if not exists test authorization dev_test;
```

## 重命名(备份)

dataname数据库为 dataname_bak

```sql
alter schema dataname rename to dataname_bak;
```

##  删除 schema(dataname)

```sql
drop schema dataname cascade;    
```

##  创建表

```sql
CREATE TABLE test."user" (
	id Integer NOT NULL,
	name Varchar(100),
	description Varchar(1024)
);
```



#  赋权

## 一个schema上的权限赋给另一个用户

```sql
GRANT USAGE ON SCHEMA dbname_dw TO dev_test;
```

## 把对某个表的操作的权限赋给另一个用户

```sql
GRANT ALL ON TABLE tw_re_pm_cell_all_cell_h to dev_test;
```

## 从某个用户收回对某个schema的使用权限

```sql
revoke all on SCHEMA dbname_dw from dev_test;
```

## 从某个用户收回对某个表的使用权限

```sql
revoke all on table fct_flux_se_flux_flow_whole_ana_d from dev_test;
```



# 序列

##  查询系统中的序列

```sql
select * from sequences;
```

## 创建序列

```sql
# 简单语法：
CREATE SEQUENCE my_seq MAXVALUE 5000 START 1;

# 标准语法：
CREATE SEQUENCE [[db-name.]schema.]sequence_name
                    ... [ INCREMENT [ BY ] positive_or_negative ]
                    ... [ MINVALUE minvalue | NO MINVALUE ]
                    ... [ MAXVALUE maxvalue | NO MAXVALUE ]
                    ... [ START [ WITH ] start ]
                    ... [ CACHE cache ]
                    ... [ CYCLE | NO CYCLE ]
```

## 使用序列

```sql
# 一个新创建还没有使用过的序列，必须首先执行NEXTVAL，然后才能执行CURRVAL。
SELECT NEXTVAL('my_seq');
SELECT CURRVAL('my_seq');
```

- 在INSERT语句里使用序列

```sql
INSERT INTO customer VALUES ('Hawkins' ,'John', 072753, NEXTVAL('my_seq'));
```

- 在INSERT语句里把序列作为默认值：

```sql
CREATE TABLE customer2(ID INTEGER DEFAULT NEXTVAL('my_seq'),
                                   lname VARCHAR(25),
                                   fname VARCHAR(25),
                                   membership_card INTEGER
                                  );
            => INSERT INTO customer2 VALUES (default,'Carr', 'Mary', 87432);
```

- 删除序列

```sql
DROP SEQUENCE seq_name;
```



#  Vertica创建外部表

```sql
CREATE EXTERNAL TABLE ext1 (x integer) AS COPY FROM '/tmp/ext1.dat' DELIMITER ',';
CREATE EXTERNAL TABLE ext1 (x integer) AS COPY FROM '/tmp/ext1.dat.bz2' BZIP DELIMITER ',';
CREATE EXTERNAL TABLE ext1 (x integer, y integer) AS COPY (x as '5', y) FROM '/tmp/ext1.dat.bz2' BZIP DELIMITER ',';
```



#  copy执行错误后的Vertica的错误日志

```shell
/database/dbname/dbname/v_dbname_node0002_catalog/CopyErrorLogs
```



#  从vertica数据的表中导出数据到数据文件

```shell
echo `vsql -d dbname -U dbadmin -Atq -w Zongfen_12 -c "select * from test.dim_flow_direction order by flow_type_code"> /database/datastage/export/dim_all/test`
```



#  通过数据文件向vertica数据库里加载数据

```shell
copy test.fct_flux_se_bus_res_ana_d from '/database/imp_file/fct_flux_se_bus_res_ana_d' on v_dbname_node0002 delimiter '|';
```



#  修改字段

##  修改字段为非空

```sql
alter table test.fct_fournet_wlanap_equp_ana_d alter column day_id set not null;
```

##  更改字段数据类型

对于数值类型：types–INTEGER, INT, BIGINT, TINYINT, INT8, SMALLINT, and all NUMERIC values of scale <=18 and precision 0 之间是可以互相转化的。此外，numeric类型的精度（precision）是无法更改的，但是长度(scale)是可以修改的，（0-18）之间可以互修改，（19-37）之间可以互修改。

```sql
 alter table test.dim_micro_area_gsm alter column cell_id set data type numeric(15,0); 
```

## 给表增加字段

```sql
 alter table test.DIM_DETAIL_SVCTYPE add column if_app numeric(10,0);
```

##  删除表字段

```sql
alter table test.DIM_DETAIL_SVCTYPE drop column if_app;
```



#  数据库表之间导数据

```shell
CONNECT TO VERTICA dbname USER dbadmin PASSWORD 'dbname' ON '192.168.1.1',5433;
export TO VERTICA dbname.test.FCT_TNES_GN_NET_M FROM test.FCT_TNES_GN_NET_M;
```



#  修改普通表为分区表

```sql
alter table test.fct_fournet_wlanap_equp_ana_d partition by day_id;
```



#  修改表名

```sql
alter table test.fct_fournet_wlanap_equp_ana_d_x rename to fct_fournet_wlanap_equp_ana_d;
```



#  修改表所属的用户

```sql
alter table test.fct_fournet_wlanap_equp_ana_d owner to dev_test
```



#  查询表

```sql
tables
```



#  projections

```sql
projections

# 查询表对应的projection
SELECT owner_name, anchor_table_name, projection_name
  FROM projections
 WHERE projection_basename = 'DIM_CFG_LEVEL';
```



#  查询列

```sql
columns
```



#  查询注释

```sql
comments

# 查询表的列对应的注释
SELECT t3.anchor_table_name AS Table_name,
       SUBSTR (t1.object_name, INSTR (t1.object_name, '.', 1) + 1) AS Column_name,
       t1.comment AS comment
FROM comments t1, projections t3
 WHERE  SUBSTR (t1.object_name, 1, INSTR (t1.object_name, '.', 1) - 1) =
              t3.projection_name
       AND t1.object_type = 'COLUMN'
ORDER BY t3.anchor_table_name;
```



#  其他操作

## 四舍五入、并且保留两位小数

```sql
SELECT TRIM (TO_CHAR (ROUND (3.456, 2.0), '999999999999999999.00')),
       TRIM (TO_CHAR (ROUND (3, 2.0), '999999999999999999.00')),
       TRIM (TO_CHAR (ROUND (3.00, 2.0), '999999999999999999.00')),
       TRIM (TO_CHAR (ROUND (323542.101, 2.0), '999999999999999999.00')),
       TRIM (TO_CHAR (ROUND (3.1067, 2.0), '999999999999999999.00'))

```

##  产生随机数

- RANDOM()

```sql
SELECT RANDOM();
```

- RANDOMINT

```sql
RANDOMINT ( N )
```

