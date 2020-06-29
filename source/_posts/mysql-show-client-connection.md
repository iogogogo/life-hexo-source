---
title: MySQL查看所有连接的客户端ip
date: 2020-06-29 13:54:14
tags: MySQL
categories: MySQL
---

有时候我们需要查看当前的mysql数据库中， 有哪些客户端保持了连接， 每个客户端分别保持了多少连接，可以使用下面的语句查询结果，可以直观的看到连接数。



```sql
SELECT substring_index(host, ':',1) AS hostname,state,count(*) FROM information_schema.processlist GROUP BY state,hostname;
```



输出结果：

```shell
mysql> SELECT substring_index(host, ':',1) AS hostname,state,count(*) FROM information_schema.processlist GROUP BY state,hostname;
+----------------+-----------------------+----------+
| hostname       | state                 | count(*) |
+----------------+-----------------------+----------+
| 10.2.1.12      |                       |        2 |
| 192.168.21.125 |                       |        2 |
| vm21122        |                       |       52 |
| localhost      | executing             |        1 |
| 192.168.21.125 | Receiving from client |        1 |
+----------------+-----------------------+----------+
5 rows in set (0.00 sec)

mysql>
```



会列出每个ip当前的状态，以及当前的连接数 。这个在处理类似碰到数据库 Too Many Connections 等的错误的时候比较有用。

