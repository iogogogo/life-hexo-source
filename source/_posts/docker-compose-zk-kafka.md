---
title: 使用docker-compose构建kafka集群
date: 2018-07-28 12:41:40
tags: docker
---



&emsp;&emsp;上篇说到了docker-compose的一些基本命令，这次使用docker-compose构建kafka集群。在说kafka之前，先简单了解一下kafka是什么东西。

## Kafka创建背景

Kafka是一个消息系统，原本开发自LinkedIn，用作LinkedIn的活动流（Activity Stream）和运营数据处理管道（Pipeline）的基础。现在它已被多家不同类型的公司作为多种类型的数据管道和消息系统使用。

活动流数据是几乎所有站点在对其网站使用情况做报表时都要用到的数据中最常规的部分。活动数据包括页面访问量（Page View）、被查看内容方面的信息以及搜索情况等内容。这种数据通常的处理方式是先把各种活动以日志的形式写入某种文件，然后周期性地对这些文件进行统计分析。运营数据指的是服务器的性能数据（CPU、IO使用率、请求时间、服务日志等等数据)。运营数据的统计方法种类繁多。

近年来，活动和运营数据处理已经成为了网站软件产品特性中一个至关重要的组成部分，这就需要一套稍微更加复杂的基础设施对其提供支持。

## Kafka简介

Kafka是一种分布式的，基于发布/订阅的消息系统。主要设计目标如下：

- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。
- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输。
- 支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。
- 同时支持离线数据处理和实时数据处理。
- Scale out：支持在线水平扩展。

### 为何使用消息系统

- 解耦

  在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

- 冗余

  

  有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

- 扩展性

  因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。

- 灵活性 & 峰值处理能力

  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

- 可恢复性

  系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

- 顺序保证

  在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。

- 缓冲

  在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。

- 异步通信

  很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

## 常用Message Queue对比

- **RabbitMQ**

  RabbitMQ是使用Erlang编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP, SMTP, STOMP，也正因如此，它非常重量级，更适合于企业级的开发。同时实现了Broker构架，这意味着消息在发送给客户端时先在中心队列排队。对路由，负载均衡或者数据持久化都有很好的支持。

- **Redis**

  Redis是一个基于Key-Value对的NoSQL数据库，开发维护很活跃。虽然它是一个Key-Value数据库存储系统，但它本身支持MQ功能，所以完全可以当做一个轻量级的队列服务来使用。对于RabbitMQ和Redis的入队和出队操作，各执行100万次，每10万次记录一次执行时间。测试数据分为128Bytes、512Bytes、1K和10K四个不同大小的数据。实验表明：入队时，当数据比较小时Redis的性能要高于RabbitMQ，而如果数据大小超过了10K，Redis则慢的无法忍受；出队时，无论数据大小，Redis都表现出非常好的性能，而RabbitMQ的出队性能则远低于Redis。

- **ZeroMQ**

  ZeroMQ号称最快的消息队列系统，尤其针对大吞吐量的需求场景。ZeroMQ能够实现RabbitMQ不擅长的高级/复杂的队列，但是开发人员需要自己组合多种技术框架，技术上的复杂度是对这MQ能够应用成功的挑战。ZeroMQ具有一个独特的非中间件的模式，你不需要安装和运行一个消息服务器或中间件，因为你的应用程序将扮演这个服务器角色。你只需要简单的引用ZeroMQ程序库，可以使用NuGet安装，然后你就可以愉快的在应用程序之间发送消息了。但是ZeroMQ仅提供非持久性的队列，也就是说如果宕机，数据将会丢失。其中，Twitter的Storm 0.9.0以前的版本中默认使用ZeroMQ作为数据流的传输（Storm从0.9版本开始同时支持ZeroMQ和Netty作为传输模块）。

- **ActiveMQ**

  ActiveMQ是Apache下的一个子项目。 类似于ZeroMQ，它能够以代理人和点对点的技术实现队列。同时类似于RabbitMQ，它少量代码就可以高效地实现高级应用场景。

- **Kafka/Jafka**

  Kafka是Apache下的一个子项目，是一个高性能跨语言分布式发布/订阅消息队列系统，而Jafka是在Kafka之上孵化而来的，即Kafka的一个升级版。具有以下特性：快速持久化，可以在O(1)的系统开销下进行消息持久化；高吞吐，在一台普通的服务器上既可以达到10W/s的吞吐速率；完全的分布式系统，Broker、Producer、Consumer都原生自动支持分布式，自动实现负载均衡；支持Hadoop数据并行加载，对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka通过Hadoop的并行加载机制统一了在线和离线的消息处理。Apache Kafka相对于ActiveMQ是一个非常轻量级的消息系统，除了性能非常好之外，还是一个工作良好的分布式系统。

## 构建zookeeper、kakfa集群

&emsp;&emsp;首先kafka依赖于zookeeper，所以在构建kafka集群之前我们先来构建zookeeper集群，zookeeper的集群构建方式很简单，docker官网都有给出docker-compose文件。zookeeper有官方镜像，kakfa暂时还没有，所以我们选择了一个github上目前star最高的镜像[**kafka-docker**](https://github.com/wurstmeister/kafka-docker)。



### 编写zookeeper的docker-compose文件

```yaml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    volumes:
      - "~/share/docker/compose-data/zookeeper/zoo1/data:/data"
      - "~/share/docker/compose-data/zookeeper/zoo1/datalog:/datalog"
      - "~/share/docker/compose-data/zookeeper/zoo1/conf:/conf"
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    volumes:
      - "~/share/docker/compose-data/zookeeper/zoo2/data:/data"
      - "~/share/docker/compose-data/zookeeper/zoo2/datalog:/datalog"
      - "~/share/docker/compose-data/zookeeper/zoo2/conf:/conf"
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    volumes:
      - "~/share/docker/compose-data/zookeeper/zoo3/data:/data"
      - "~/share/docker/compose-data/zookeeper/zoo3/datalog:/datalog"
      - "~/share/docker/compose-data/zookeeper/zoo3/conf:/conf"
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```

&emsp;&emsp;保存文件名为【docker-compose.yml】,我们可以看到整个compose文件中一共运行了三个zookeeper实例，为什么是三个？因为zookeeper内部的选举机制，在集群中有其他节点挂掉的话，至少保证n+1个节点用来选举leader，所以集群至少三个节点。

&emsp;&emsp;现在来说说compose文件中的参数含义：

1. version: '3.1'  表示使用第三代compose语法
2. services 表示一个实例服务
3. zoo1、zoo2、zoo3是我自己给实例服务起的名字
4. image 表示需要使用的docker镜像，这里使用的官方镜像，并且没有指定target
5. restart: always 表示挂掉会一直重启
6. ports 导出的端口号，因为我这是在一台机器，所以分别使用了2181，2182，2183三个端口
7. volumes 数据卷目录
8. environment 环境参数，这里面ZOO_MY_ID表示集群中的zk编号，不可重复，ZOO_SERVERS表示集群中所有的zk服务实例

### 编写kakfa的docker-compose文件

kakfa的compose文件和zookeeper有些小区别，主要体现在运行方式上面并且我们现在需要创建三个docker-compose文件，唯一的区别就是port不一样。需要注意的是environment参数中的IP地址不要使用localhost或者127.0.0.1，这里需要使用IP地址

- docker-compose-9092.yml

```yaml
version: '3.1'

services:
  kafka:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka
    ports:
      - 9092:9092
    volumes:
      - "~/share/docker/compose-data/kafka/kafka-9092/docker.sock:/var/run/docker.sock"
      - "~/share/docker/compose-data/kafka/kafka-9092:/kafka"
    environment:
      KAFKA_VERSION: 1.1.0
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.6
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: 192.168.1.6:2181,192.168.1.6:2182,192.168.1.6:2183
```

- docker-compose-9093.yml

```yaml
version: '3.1'

services:
  kafka:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka
    ports:
      - 9093:9092
    volumes:
      - "~/share/docker/compose-data/kafka/kafka-9093/docker.sock:/var/run/docker.sock"
      - "~/share/docker/compose-data/kafka/kafka-9093:/kafka"
    environment:
      KAFKA_VERSION: 1.1.0
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.6
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ZOOKEEPER_CONNECT: 192.168.1.6:2181,192.168.1.6:2182,192.168.1.6:2183
```

- docker-compose-9094.yml

```yaml
version: '3.1'

services:
  kafka:
    image: wurstmeister/kafka
    restart: always
    hostname: kafka
    ports:
      - 9094:9092
    volumes:
      - "~/share/docker/compose-data/kafka/kafka-9094/docker.sock:/var/run/docker.sock"
      - "~/share/docker/compose-data/kafka/kafka-9094:/kafka"
    environment:
      KAFKA_VERSION: 1.1.0
      KAFKA_ADVERTISED_HOST_NAME: 192.168.1.6
      KAFKA_ADVERTISED_PORT: 9094
      KAFKA_ZOOKEEPER_CONNECT: 192.168.1.6:2181,192.168.1.6:2182,192.168.1.6:2183
```

### 运行zk与kakfa集群

&emsp;&emsp;到这里，我们已经把两个组件的docker-compose文件写好了，现在我们来运行集群。

#### 运行zookeeper集群

在zookeeper的docker-compose文件夹下面打开终端，运行docker-compose up，这里也可以使用docker-compose up -d 后台运行

```shell
➜  zk-kafka docker-compose up
Creating network "zk-kafka_default" with the default driver
Pulling zoo1 (zookeeper:)...
latest: Pulling from library/zookeeper
Digest: sha256:a043534003831de15268779b582d407d37291bf7d22292ec7dce242c57a5a2be
Status: Downloaded newer image for zookeeper:latest
Creating zk-kafka_zoo3_1 ... done
Creating zk-kafka_zoo2_1 ... done
Creating zk-kafka_zoo1_1 ... done
Attaching to zk-kafka_zoo1_1, zk-kafka_zoo3_1, zk-kafka_zoo2_1
zoo1_1  | ZooKeeper JMX enabled by default
zoo3_1  | ZooKeeper JMX enabled by default
zoo1_1  | Using config: /conf/zoo.cfg
zoo3_1  | Using config: /conf/zoo.cfg
zoo2_1  | ZooKeeper JMX enabled by default
zoo2_1  | Using config: /conf/zoo.cfg
zoo1_1  | log4j:WARN No appenders could be found for logger (org.apache.zookeeper.server.quorum.QuorumPeerConfig).
zoo1_1  | log4j:WARN Please initialize the log4j system properly.
zoo1_1  | log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
zoo3_1  | log4j:WARN No appenders could be found for logger (org.apache.zookeeper.server.quorum.QuorumPeerConfig).
zoo3_1  | log4j:WARN Please initialize the log4j system properly.
zoo3_1  | log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
zoo2_1  | log4j:WARN No appenders could be found for logger (org.apache.zookeeper.server.quorum.QuorumPeerConfig).
zoo2_1  | log4j:WARN Please initialize the log4j system properly.
zoo2_1  | log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

#### 查看zookeeper的运行情况

新开终端，执行docker ps，可以看到，我们已经有三个zookeeper实例已经运行起来了

```shell
➜  ~ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                        NAMES
7278fca16a38        zookeeper           "/docker-entrypoint.…"   24 seconds ago      Up 27 seconds       2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp   zk-kafka_zoo1_1
bc396f78a9d4        zookeeper           "/docker-entrypoint.…"   24 seconds ago      Up 27 seconds       2888/tcp, 3888/tcp, 0.0.0.0:2183->2181/tcp   zk-kafka_zoo3_1
dd9b8be8eebe        zookeeper           "/docker-entrypoint.…"   24 seconds ago      Up 26 seconds       2888/tcp, 3888/tcp, 0.0.0.0:2182->2181/tcp   zk-kafka_zoo2_1
```

#### 运行kakfa

在kakfa的docker-compose-909*.yml文件夹下面分布执行下面三个命令

```shell
# 集群启动方式
docker-compose -f docker-compose-9092.yml up -d
docker-compose -f docker-compose-9093.yml scale kafka=2
docker-compose -f docker-compose-9094.yml scale kafka=3
```

#### 查看kafka的运行情况

docker ps 可以看到现在已经有三个zookeeper实例和三个kafka实例

```shell
➜  ~ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                                        NAMES
31e7e8ad1fc5        wurstmeister/kafka   "start-kafka.sh"         29 seconds ago      Up 37 seconds       0.0.0.0:9094->9092/tcp                       zk-kafka_kafka_3
d9c1fa25f46e        wurstmeister/kafka   "start-kafka.sh"         35 seconds ago      Up 43 seconds       0.0.0.0:9093->9092/tcp                       zk-kafka_kafka_2
caf6b21ba709        wurstmeister/kafka   "start-kafka.sh"         44 seconds ago      Up 52 seconds       0.0.0.0:9092->9092/tcp                       zk-kafka_kafka_1
7278fca16a38        zookeeper            "/docker-entrypoint.…"   37 minutes ago      Up About a minute   2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp   zk-kafka_zoo1_1
bc396f78a9d4        zookeeper            "/docker-entrypoint.…"   37 minutes ago      Up About a minute   2888/tcp, 3888/tcp, 0.0.0.0:2183->2181/tcp   zk-kafka_zoo3_1
dd9b8be8eebe        zookeeper            "/docker-entrypoint.…"   37 minutes ago      Up About a minute   2888/tcp, 3888/tcp, 0.0.0.0:2182->2181/tcp   zk-kafka_zoo2_1

```

### 使用kafka集群进行消息生产与消费

#### 进入kafka容器

docker exec -it [container-name] /bin/bash，这里使用的是容器的名称，因为有三个kafka集群，通过docker ps可以看到分别是（zk-kafka_kafka_1，zk-kafka_kafka_2，zk-kafka_kafka_3）所以随便一个都可以

```shell
# 进入kafka容器
docker exec -it zk-kafka_kafka_1 /bin/bash

# 进入安装目录
cd /opt/kafka/

# 查看topic列表
./bin/kafka-topics.sh --list --zookeeper 192.168.1.6:2181

# 创建 topic
./bin/kafka-topics.sh --create --zookeeper 192.168.1.6:2181 --replication-factor 1 --partitions 1 --topic test

# 生产消息
./bin/kafka-console-producer.sh --broker-list 192.168.1.6:9092 --topic test

# 消费消息
./bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.6:9092 --topic test --from-beginning
```

##### 查看topic列表

```shell
➜  ~ docker exec -it zk-kafka_kafka_1 /bin/bash
bash-4.4# cd /opt/kafka
bash-4.4# pwd
/opt/kafka
bash-4.4# ./bin/kafka-topics.sh --list --zookeeper 192.168.1.6:2181
bash-4.4#
```

##### 创建topic

```shell
➜  ~ docker exec -it zk-kafka_kafka_1 /bin/bash
bash-4.4# cd /opt/kafka
bash-4.4# pwd
/opt/kafka
bash-4.4# ./bin/kafka-topics.sh --list --zookeeper 192.168.1.6:2181
ion-factor 1 --partitions 1 --topic test --zookeeper 192.168.1.6:2181 --replicati
Created topic "test".
bash-4.4#
```

##### 消息的生产与消费

打开两个终端进入容器，一个作为生产者生产消息，一个作为消费者消费消息

- 生产者使用zk-kafka_kafka_1容器，zk使用9092端口

```shell
➜  zk-kafka docker exec -it zk-kafka_kafka_1 /bin/bash
bash-4.4# cd /opt/kafka
bash-4.4# ./bin/kafka-console-producer.sh --broker-list 192.168.1.6:9092 --topic test
>123
>测试数据
>test data
>小花脸
>
```

- 消费者zk-kafka_kafka_2，zk使用9093端口

```shell
➜  ~ docker exec -it zk-kafka_kafka_2 /bin/bash
bash-4.4# cd /opt/kafka
bash-4.4# ./bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.6:9093 --topic test --from-beginning
123
测试数据
test data
小花脸
```
- 消费者zk-kafka_kafka_3，zk使用9094端口

```shell
➜  ~ docker exec -it zk-kafka_kafka_3 /bin/bash
bash-4.4# cd /opt/kafka
bash-4.4# ./bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.6:9094 --topic test --from-beginning
123
测试数据
test data
小花脸
```
至此，zookeeper和kakfa集群搭建到此结束

## 小结

中间踩了很多坑，网上参考了很多教程，重点还是需要自己动手折腾



## 参考

[docker从入门到实践](https://yeasy.gitbooks.io/docker_practice/compose/compose_file.html)

[kakfa-docker](https://github.com/wurstmeister/kafka-docker)

[Docker部署Kafka集群](https://www.guonanjun.com/268.html)

[zookeeper-docker](https://hub.docker.com/_/zookeeper/)