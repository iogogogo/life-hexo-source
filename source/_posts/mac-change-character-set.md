---
title: Mac 修改MySQL编码集
date: 2019-03-09 22:05:06
tags: Mac
---

前两天因为Cisco的VPN坏了，一直用不了，所以重新还原了Mac，重新安装了所有的软件，其中就有MySQL-5.7.25。之前安装以后也是第一时间设置了MySQL的编码为utf8，这次新安装也再次记录一下修改步骤

## 查看Mac 安装MySQL以后的默认编码

```shell
SHOW VARIABLES LIKE 'character_set_%';

# 结果
character_set_client	utf8mb4
character_set_connection	utf8mb4
character_set_database	latin1 ## 这里是默认编码
character_set_filesystem	binary
character_set_results	utf8mb4
character_set_server	latin1  ## 这里是默认编码
character_set_system	utf8
character_sets_dir	/usr/local/mysql-5.7.25-macos10.14-x86_64/share/charsets/
```

## 新建my.cnf文件

```shell
[client]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

[mysql]
default-character-set=utf8
```

## 移动到 /etc 目录

移动my.cnf文件到/etc目录，需要root权限

```shell
sudo mv my.cnf /etc/
```

## 重启MySQL，再次查看编码

```shell
SHOW VARIABLES LIKE 'character_set_%';

# 结果
character_set_client	utf8mb4
character_set_connection	utf8mb4
character_set_database	utf8 ## 这里是修改以后的
character_set_filesystem	binary
character_set_results	utf8mb4
character_set_server	utf8 ## 这里是修改以后的
character_set_system	utf8
character_sets_dir	/usr/local/mysql-5.7.25-macos10.14-x86_64/share/charsets/
```
