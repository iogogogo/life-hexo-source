---
title: Redis查看集群信息以及key分布非slot
date: 2020-07-29 16:08:34
tags: Redis
categories: Redis
---



## 进入redis集群

```shell
redis-cli -h ip -p port -c
```



## 查看集群节点

```shell
cluster nodes
```



## 查看key对应的slot

```shell
cluster keyslot key
```



## 查看slot和节点的对应关系

```shell
cluster slots
```

