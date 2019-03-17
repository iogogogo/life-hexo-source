---
title: Vertica 分析函数
date: 2019-03-14 11:33:07
tags: Vertica
---

分析函数作为 SQL 言语的一种扩展已经被纳入了美国国家标准化组织SQL 委员会的SQL 规范说明书中。所以不同数据库厂商支持的分析函数其语法结构和函数名称也基本一致。这里仅介绍Vertica的的分析函数语法和函数作用，应用函数相关例子略。

# 分析函数语法

```sql
ANALYTIC_FUNCTION（argument-1，...，argument-n）
OVER（[window_partition_clause] 
[window_order_clause] 
[window_frame_clause）
```

## 常见函数用途和描述

| 和              | 该函数计算组中表达式的累积和                                 |
| --------------- | ------------------------------------------------------------ |
| MIN             | 在一个组中的数据窗口中查找表达式的最小值                     |
| MAX             | 在一个组中的数据窗口中查找表达式的最大值                     |
| AVG             | 用于计算一个组和数据窗口内表达式的平均值。                   |
| 计数            | 对一组内发生的事情进行累积计数                               |
| 秩              | 根据ORDER BY子句中表达式的值，从查询返回的每一行，计算它们与其它行的相对位置 |
| DENSE_RANK      | 根据ORDER BY子句中表达式的值，从查询返回的每一行，计算它们与其它行的相对位置 |
| FIRST_VALUE     | 返回组[中数据](https://www.baidu.com/s?wd=%E4%B8%AD%E6%95%B0%E6%8D%AE&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)窗口的第一个值 |
| LAST_VALUE      | 返回组中数据窗口的最后一个值。                               |
| 落后            | 可以访问结果集中的其它行而不用进行自连接                     |
| 铅              | LEAD与LAG相反，LEAD可以访问组中当前行之后的行                |
| ROW_NUMBER      | 返回有序组中一行的偏移量，从而可用于按特定标准排序的行号     |
| STDDEV          | 计算当前行关于组的标准偏离                                   |
| STDDEV_POP      | 该函数计算总体标准偏离，并返回总体变量的平方根               |
| STDDEV_SAMP     | 该函数计算累积样本标准偏离，并返回总体变量的平方根           |
| VAR_POP         | 该函数返回非空集合的总体变量（忽略null）                     |
| VAR_SAMP        | 该函数返回非空集合的样本变量（忽略null）                     |
| 方差            | 如果表达式[中行](https://www.baidu.com/s?wd=%E4%B8%AD%E8%A1%8C&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)数为1，则返回0，如果表达式中行数大于1，则返回VAR_SAMP |
| CUME_DIST       | 计算一行在组中的相对位置                                     |
| NTILE           | 将一个组分为“表达式”的散列表示                               |
| PERCENT_RANK    | 和CUME_DIST（累积分配）函数类似                              |
| PERCENTILE_DISC | 返回一个与输入的分布百分比值相对应的数据值                   |
| PERCENTILE_CONT | 返回一个与输入的分布百分比值相对应的数据值                   |
| MEDIAN          | 在一个组中的数据窗口中查找最小值与最大值的平均值             |

#  参数-1，...，参数-n 

分析函数的参数



#  window_partition_clause 

根据划分表达式设置的规则， PARTITION BY （按... 划分）将一个结果逻辑分成Ñ 个分组划分表达式。在此“ 划分” 和“ 分组” 用作同义词。分析函数独立应用于各个分组，并在应用时重置。

```sql
OVER（PARTITION BY expression [，...]）
```



#  window_order_clause 

ORDER BY （按... 排序）语句规定了每个分组（划分）的数据如何排序。这些必然影响分析函数的结果。

```sql
OVER（ORDER BY expression [{ASC | DESC}] 
... [NULLS {FIRST | LAST | AUTO}] [，expression ...]）
```



#  window_frame_clause 

窗口生成语句用以定义滑动或固定数据窗口，分析函数在分组内进行分析。该语句能够对分组中任意定义的滑动或固定窗口进行计算。

