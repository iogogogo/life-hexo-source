---
title: Hadoop常用运维命令
date: 2020-08-03 17:44:57
tags: Hadoop
categories: Hadoop
---

## 一、HADOOP集群启动
### 1. 格式化zk

在zk的leader节点服务器上，Hadoop的bin目录中执行如下命令：
改为在 nn的active节点上
```
sh hdfs zkfc -formatZK
```
### 2. 启动journalnode集群
hadoop任意节点服务器执行
```
hadoop-daemons.sh start journalnode
```
### 3. 格式化namenode
在nn节点执行
```
hadoop namenode -format
```
### 4. 启动NameNode
在hdaoop01节点上执行如下命令，启动NameNode节点：
```
hadoop-daemon.sh start namenode
```
首先把hdaoop02服务器的 namenode节点变为standby namenode节点。
执行命令如下：
```
hdfs namenode -bootstrapStandby
```
启动hadoop02服务器的namenode节点，执行命令如下：
```
hadoop-daemon.sh start namenode
```
### 5. 启动DataNode
在hadoop01,hadoop02,hadoop03服务器上分别启动datanode节点，在这三台服务器上分别执行如下命令：
```
hadoop-daemon.sh start datanode
```
### 6. 启动zkfc
FalioverControllerActive是失败恢复线程。这个线程需要在NameNode节点所在的服务器上启动，在hadoop01,hadoop02服务器上执行如下命令：
```
hadoop-daemon.sh start zkfc
```
### 7. 启动Resourcemanager
在**hdaoop01**服务器上启动主Resourcemanager节点，执行如下命令：   
启动成功后，hadoop01,hadoop02,hadoop03服务器上的nodemanager 也会跟随启动
```
start-yarn.sh
```
在**hadoop02**服务器上启动副 Resoucemanager节点，执行如下命令：
```
yarn-daemon.sh start resourcemanager
```
## 二、YARN运维命令
### 1. yarn application
1、-list 列出所有 application 信息
```
yarn application -list
```
2、-appStates <States>跟-list一起使用，用来筛选不同状态的application，多个用","分隔；所有状态ALL,NEW,NEW_SAVING,SUBMITTED,ACCEPTED,RUNNING,FINISHED,FAILED,KILLED
```
yarn application -list -appStates RUNNING
```
3、-appTypes <Types>跟-list一起使用，用来筛选不同类型的application，多个用","分隔；如 MAPREDUCE
```
yarn application -list -appTypes MAPREDUCE
```
4、-kill <Application ID>杀死一个application，需要指定一个Application ID
```
yarn  application -kill application_1526100291229_206393
```
5、-status <Application ID>列出 某个application 的状态
```
yarn application -status application_1526100291229_206393
```
6、-movetoqueue <ApplicationID>移动application到其他的queue，不能单独使用
7、-queue <Queue Name> 与 movetoqueue命令一起使用，指定移动到哪个queue
```
yarn application -movetoqueue application_1526100291229_206393 -queue other
```
### 2. yarn node
查看各个node上的任务数
```
yarn  node --list
```
### 3. yarn logs
```
yarn logs -applicationId application_1583405966138_0013
```