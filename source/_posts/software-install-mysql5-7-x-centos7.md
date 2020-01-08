---
title: CentOS7 安装 MySQL5.7.X
date: 2020-01-08 19:31:55
tags: MySQL
categories: MySQL
---



## **安装前准备**

MySQL安装需要准备root账户



### 卸载MariaDB相关组件

> 由于MariaDB与MySQL类似，在安装时候会提示与已经安装的RPM包有冲突，因此需要卸载一些包含有MariaDB关键字的RPM包。 
>
> 执行`rpm -qa | grep mysql`检查需要卸载的包
>
>  执行`rpm -qa | grep mariadb`检查需要卸载的包
>
> 如发现存`mariadb`与`mysql`在则使用`rpm -e --nodeps xxx`进行卸载

```SHELL
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -qa | grep mariadb # 检查mariadb
mariadb-libs-5.5.64-1.el7.x86_64

# 如有就进行卸载
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
```
### 下载安装文件

这里我直接在官网拿到了MySQL5.7.28的下载地址，通过wget进行下载

```shell
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```



### 解压安装文件

这里解压文件以后，直接放到新文件夹`mysql-5.7.28-1.el7.x86_64`了，可以看到目录下面有很多rpm包

```verilog
tar xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar -C mysql-5.7.28-1.el7.x86_64

[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# ll
总用量 595272
-rw-r--r-- 1 7155 31415  45109364 9月  30 16:04 mysql-community-client-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415    318768 9月  30 16:04 mysql-community-common-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415   7037096 9月  30 16:04 mysql-community-devel-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415  49329100 9月  30 16:04 mysql-community-embedded-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415  23354908 9月  30 16:04 mysql-community-embedded-compat-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415 136837816 9月  30 16:04 mysql-community-embedded-devel-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415   4374364 9月  30 16:04 mysql-community-libs-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415   1353312 9月  30 16:04 mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415 208694824 9月  30 16:05 mysql-community-server-5.7.28-1.el7.x86_64.rpm
-rw-r--r-- 1 7155 31415 133129992 9月  30 16:05 mysql-community-test-5.7.28-1.el7.x86_64.rpm
```





## 开始安装

### 检查3306端口

MySQL默认使用3306端口，安装前查看端口是否被占用

```shell
netstat -anp|grep 3306
```



### 安装相应的rpm包

- mysql-community-common-5.7.28-1.el7.x86_64.rpm
- mysql-community-libs-5.7.28-1.el7.x86_64.rpm
- mysql-community-client-5.7.28-1.el7.x86_64.rpm
- mysql-community-server-5.7.28-1.el7.x86_64.rpm

以上四个文件按照顺序依次安装，顺序不能错乱

```shell
rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```



以下为安装日志，只要之前的`mariadb`和`mysql`卸载干净了，一般不会有问题

```verilog
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-common-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-5.7.28-1.e################################# [100%]
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-libs-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-5.7.28-1.el7################################# [100%]
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-client-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-5.7.28-1.e################################# [100%]
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.28-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-5.7.28-1.e################################# [100%]
```



## 后续工作

### 启动MySQL服务

```shell
systemctl start mysqld
```

### 查看MySQL服务状态

```shell
systemctl status mysqld
```

输出日志

```verilog
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# systemctl status mysqld
Redirecting to /bin/systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 三 2020-01-08 20:21:12 CST; 35s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 19471 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 19408 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 19474 (mysqld)
    Tasks: 27
   Memory: 324.4M
   CGroup: /system.slice/mysqld.service
           └─19474 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

1月 08 20:21:04 vm31_123 systemd[1]: Starting MySQL Server...
1月 08 20:21:12 vm31_123 systemd[1]: Started MySQL Server.
```

### 修改默认密码

#### 查看MySQL安装成功以后生成的零时密码

```shell
sudo cat /var/log/mysqld.log |grep 'temporary password'
```

输出结果，可以看到我们的mysql零时密码是`;2o1:2h%!ruT`

```verilog
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# sudo cat /var/log/mysqld.log |grep 'temporary password'
2020-01-08T12:21:09.914526Z 1 [Note] A temporary password is generated for root@localhost: ;2o1:2h%!ruT
```

#### 使用零时密码登录修改

输入`mysql -uroot -p`以后会提示我们输入密码，这时候输入刚刚生成的零时密码`;2o1:2h%!ruT`即可登录

接下来就可以使用`ALTER USER 'root'@'localhost' IDENTIFIED BY 'MySQL@123';`（此处MySQL@123为数据库密码，根据需求自行设置更改）

```verilog
[root@vm31_123 mysql-5.7.28-1.el7.x86_64]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.28

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MySQL@123';
Query OK, 0 rows affected (0.00 sec)

mysql>
```

## 设置外部IP访问MySQL

为了能够让外部访问数据库，需要将root的Host值改为'%'，具体步骤如下：

查看允许连接到本数据库的信息，执行命令`select host,user from mysql.user;`

默认root只能通过localhost连接，不能远程访问。

- 选择数据库：`use mysql;`
- 先执行：`update user set Host='%' where User='root';`
-  再执行:`flush privileges;` # 刷新权限

```verilog
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set Host='%' where User='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
```

当设置完host=%以后，就可以使用我们熟悉`navicat`或者`sequel`进行连接了

## 修改MySQL默认编码为`utf8`

当我们MySQL安装完成以后，默认编码不一定是`utf8`，虽然在建库建表的时候可以指定，但是始终不方便，那么接下来我们将MySQL默认编码设置为`utf8`

### 查看MySQL默认编码

使用`show variables like 'character_set_%';`

```verilog
mysql> show variables like 'character_set_%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

可以看到`character_set_server`这里是`latin1`，并不是我们想要的`utf8`，这时候我们就需要编辑`/etc/my.cnf`文件进行设置

### 编辑`my.cnf`文件

`sudo vim /etc/my.cnf`进行编辑，加入以下内容，然后保存重启MySQL服务`systemctl restart mysqld`

**注意：`my.cnf`中默认包含`[mysqld]`，修改的时候就在原来的基础上改就好了，不要重复了**



```verilog
[client]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

[mysql]
default-character-set=utf8
```



### 再次查看修改以后的编码

```verilog
mysql> show variables like 'character_set_%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```



到这里，MySQL的默认编码就是我么熟悉的`utf8`了



## 忘记MySQL密码

当忘记MySQL密码或者MySQL密码错误是会提示错误`ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)`

如果找不到MySQL密码时可以使用跳过mysql权限表启动的方式进行密码重置

1. 停止MySQL服务

   ```shell
   systemctl stop mysqld
   ```

2. 跳过mysql权限表启动

   ```shell
   /usr/bin/mysqld_safe --skip-grant-tables
   ```

3. 无密码登录

   ```shell
   mysql -uroot
   ```

   

4. 选择`mysql`数据库

   ```sql
   use mysql; 
   ```

   

5. 重置密码

   ```sql
   update user set authentication_string=password('MySQL@123') where user='root'; 
   ```

   

6. 刷新权限

   ```shell
   flush privileges;
   ```



## 常见问题



1. 修改默认配置，当插入或更新的数据比较大时，需要修改`/etc/my.cnf`配置文件

   在[mysqlId]下加上 max_allowed_packet=20M (可以通过 mysql --help | grep my.cnf查找文件路径)



2. 当重启服务器后，MySQL无法正常启动，遇到以下问题 `Job for mysqld.service failed. See 'systemctl status mysqld.service' and 'journalctl -xn' for details.` 查看日志，出现如下错误内容：

   ```verilog
   [ERROR] /usr/sbin/mysqld: Can't create/write to file '/var/run/mysqld/mysqld.pid' (Errcode: 2 - No such file or directory)
   [ERROR] Can't start server: can't create PID file: No such file or directory
   Copy
   ```

   解决方案：

   ```
   ##授权
   chown mysql.mysql /var/run/mysqld/
   
   ##启动
   /etc/init.d/mysqld start
   ```
   
