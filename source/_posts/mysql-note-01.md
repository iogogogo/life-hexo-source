---
title: MySQL 常用语句
date: 2019-03-14 16:21:43
tags: MySQL
---

# MySQL 常用语句

[sql-join](http://www.powerxing.com/sql-join/)
[mysql常用sql语句总结](https://mp.weixin.qq.com/s?__biz=MzAwOTE3NDY5OA==&mid=2647906520&idx=2&sn=e1da610a41a0692721d3756f9e1583bb&chksm=8344d31db4335a0bf5164b6c57c7e49f6a89f0538d778b156450bbaceba55e6a47abcebf2dd1&mpshare=1&scene=1&srcid=0128OHhJN5V4SPLdlN1uN4lb#rd)



## 修改替换字段中的某一个值

```sql
update dictionary set content=REPLACE(content, '谭', '谈');
```
```sql
update recipes set item_image=REPLACE(item_image,'http://app-file.botu.com:9000','http://app-file.botu.com:9000/BOTU')
where id in (select rec.id from (select id from recipes r where  binary r.item_image not like '%BOTU%') rec);
```

## 清空表数据

```sql
truncate table dictionary; -- 清空自增主键
delete from dictionary;	-- 不清空自增主键
```

## NULL查询

```sql
select * from tmp where name is not null;
select * from tmp where name is null;
```

## 根据表结构生成实体对应字段

```sql
SELECT
    CONCAT_WS(
        '',
        '/**
         *',
         COLUMN_COMMENT,
         '
         */
         @TableField("',column_name,'")
         @JSONField(name = "',column_name,'")
         private ',
        CASE DATA_TYPE
    WHEN 'varchar' THEN
        'String '
    WHEN 'bigint' THEN
        'Long '
    WHEN 'longtext' THEN
        'String '
    WHEN 'datetime' THEN
        'Date '
    WHEN 'int' THEN
        'Integer '
    WHEN 'decimal' THEN
        'BigDecimal '
    WHEN 'double' THEN
        'Double '
    WHEN 'timestamp' THEN
        'Timestamp '
    WHEN 'longblob' THEN
        'Byte[] '
    WHEN 'tinyint' THEN
        'Integer '
    WHEN 'text' THEN
        'String '
    WHEN 'char' THEN
        'Char '
    WHEN 'date' THEN
        'Date '
    WHEN 'float' THEN
        'Float '
    WHEN 'varbinary' THEN
        'Byte[] '
    WHEN 'mediumtext' THEN
        'String '
    WHEN 'enum' THEN
        'String '
    WHEN 'blob' THEN
        'Byte[] '
    WHEN 'set' THEN
        'String '
    WHEN 'time' THEN
        'Time '
    WHEN 'smallint' THEN
        'Integer '
    WHEN 'tinytext' THEN
        'String '
    WHEN 'binary' THEN
        'Byte[] '
    WHEN 'bit' THEN
        'Boolean '
    END,
    column_name,
    ';'
    )
FROM
    information_schema.`COLUMNS`
WHERE
    TABLE_SCHEMA = 'life_health'
AND TABLE_NAME = 'health_exp'; 
```

## 表操作

### 增加与修改列

```sql
Alter table 表名 add 列名称 列类型 列参数; [加的列在表的最后]
例: alter table m1 add birth date not null default '0000-00-00';

Alter table 表名 add 列名称 列类型 列参数 after 某列; [把新列加在某列后]
例: alter table m1 add gender char(1) not null default '' after username;

Alter table 表名 add 列名称 列类型 列参数 first; [把新列加在最前面]
例: alter table m1 add pid int not null default 0 first;
```

### 删除列

```sql
Alter table 表名 drop 列名;
```

### 修改列类型

```sql
Alter table 表名 modify 列名 新类型 新参数; (不能修改列名);
例:alter table m1 modify gender char(4) not null default '';
```

### 修改列名及列类型

```sql
Alter table 表名 change 旧列名 新列名 新类型 新参数;
例:alter table m1 change id uid int unsigned;
```

### 改表名

```sql
rename table regist3 to reg3;
```

### 删除表

```sql
DROP TABLE IF EXISTS `dictionary`;
```

### 常用查询

```sql
SELECT * FROM sensitive_word_condition a WHERE a.count < 11 ORDER BY a.count DESC LIMIT 1;
```

# MySQL 5.7查询一段时间内最后一条数据

mysql 升级到5.7之后，存储引擎做了一些优化，之前我们使用的 先order by 再 group by 的方式取最后一条数据的查询，会出现取的不是最后一条记录的问题

为了实现原有查询逻辑，请在子查询后面，加上limit 9223372036854775807
其中，9223372036854775807为bigint的最大值

```sql
SELECT
	*
FROM
	(
		SELECT
			e.*,
			DATE_FORMAT(`update_date`, '%Y-%m-%d') AS edt
		FROM
			health_exp e
		WHERE
			update_date BETWEEN '2018-02-01'
		AND '2018-02-06 23:23:59'
		AND user_id = '2baef230bad144e89296c2c51fa2b680'
		ORDER BY
			update_date DESC
		LIMIT 9223372036854775807
	) t
GROUP BY
	t.edt;
```

