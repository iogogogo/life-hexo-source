---
title: Spring Boot 整合kafka
date: 2020-01-06 23:28:32
tags:
 - Kafka
 - Spring Boot
cover: true
categories: Spring Boot
---

关于kafka的介绍这里就不在过多说明，可以看之前写过一遍文章[使用docker-compose构建kafka集群](http://localhost:4000/2018/07/28/docker-compose-zk-kafka/) 文章里面有关于kafka的一些介绍以及环境搭建，文章中的环境搭建是基于docker和docker-compose的，如果不想通过docker构建，也可以直接下载kafka的安装包直接在机器上启动，之前的文章链接[kafka常用操作笔记](http://localhost:4000/2018/07/30/kafka-note/)。这里也不再叙述，今天主要来看Spring Boot中如何对接kafka进行数据的消费与生产。



## 配置POM

第一步当然是先引入pom依赖

```xml
 <dependency>
   <groupId>org.springframework.kafka</groupId>
   <artifactId>spring-kafka</artifactId>
</dependency>
```



## 配置kafka基本信息

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092  # 多个使用`,`隔开
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      retries: 0 # 失败重试次数
      batch-size: 16384
      buffer-memory: 33554432
      acks: -1 # 取值 all, -1, 0, 1
    consumer:
      enable-auto-commit: true
      auto-commit-interval: 5000
      group-id: group-test
      auto-offset-reset: earliest # 消费offset取值earliest,latest,none（默认：latest）
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
# 自定义producer topic
kafka:
  producer:
    topic: test-producer
```

详细的配置介绍：[[spring-kafka生产者消费者配置详解](https://www.cnblogs.com/yx88/p/11013338.html)](https://www.cnblogs.com/yx88/p/11013338.html)



## 启动zookeeper和kafka服务

### 启动zookeeper

```shell
./bin/zookeeper-server-start.sh  -daemon  config/zookeeper.properties
```

### 启动kafka

```shell
./bin/kafka-server-start.sh -daemon config/server.properties
```



通过`jps`检查进程

```shell
➜  kafka_2.11-2.3.0 jps
27442
40473 QuorumPeerMain # zookeeper进程
40795 Kafka # kafka进程
40847 Jps
```



## 生产消息

为了方便看数据，我们定义一个`Metric`类，用来保存数据，并每隔`3s`往kafka服务器发送一次数据，并且在程序启动以后，通过`CommandLineRunner`初始化发送

- metric类

```java
import com.google.gson.annotations.SerializedName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.Map;

/**
 * Created by tao.zeng on 2020-01-07.
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Metric implements Serializable {

    private String hostname;

    private long total;

    @SerializedName("succ_cnt")
    private long succCnt;

    @SerializedName("succ_rate")
    private float succRate;

    private int status;

    private LocalDateTime timestamp;

    private Map<String, Object> tags;
}

```

- Spring Boot启动类

```java
import com.google.common.collect.Maps;
import com.iogogogo.common.util.DecimalUtils;
import com.iogogogo.common.util.IdHelper;
import com.iogogogo.common.util.JsonParse;
import com.iogogogo.kafka.pojo.Metric;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.kafka.core.KafkaTemplate;

import java.time.LocalDateTime;
import java.util.Map;
import java.util.Random;
import java.util.concurrent.TimeUnit;

/**
 * Created by tao.zeng on 2020-01-06.
 * 动态配置多个topic
 * https://github.com/spring-projects/spring-kafka/issues/361
 */
public class KafkaApplication implements CommandLineRunner {

    @Autowired
    private KafkaTemplate kafkaTemplate;

    /**
     * 这里对应我们在yml中自定义的配置，用于获取发送数据用的topic
     */
    @Value("${kafka.producer.topic}")
    private String producerTopic;

    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        Random random = new Random();
        while (true) {
            writer2Kafka(random);
            // 每隔3s向kafka发送数据
            TimeUnit.SECONDS.sleep(3);
        }
    }

    private void writer2Kafka(Random random) {
        Metric metric = new Metric();
        metric.setHostname("NODE-" + IdHelper.id());
        metric.setTotal(random.nextInt(10000));
        metric.setSuccCnt(random.nextInt(9900));

        float v = DecimalUtils.divide(metric.getSuccCnt(), metric.getTotal());
        metric.setSuccRate(v);
        metric.setStatus(random.nextInt(2));
        metric.setTimestamp(LocalDateTime.now());
        Map<String, Object> tagMap = Maps.newHashMap();
        tagMap.put("cpu_util", random.nextFloat() * 100);
        tagMap.put("mem_util", random.nextFloat() * 100);
        metric.setTags(tagMap);

        kafkaTemplate.send(producerTopic, JsonParse.toJson(metric));
        kafkaTemplate.flush();
    }
}

```

启动程序后，如果topic不存在则会自动创建topic（我们并没有关闭自动创建topic），通过kafka-consumeer命令可以看到topic中的数据

```shell
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-producer --from-beginning

# 样例数据
{"hostname":"NODE-55","total":6704,"succ_cnt":1441,"succ_rate":0.2149463,"status":0,"timestamp":"2020-01-07T21:29:21.501","tags":{"mem_util":5.2691402,"cpu_util":97.53448}}
{"hostname":"NODE-23","total":9579,"succ_cnt":2191,"succ_rate":0.22872952,"status":0,"timestamp":"2020-01-07T21:29:25.422","tags":{"mem_util":27.203756,"cpu_util":43.626713}}
{"hostname":"NODE-8","total":6889,"succ_cnt":786,"succ_rate":0.114094935,"status":0,"timestamp":"2020-01-07T21:29:28.428","tags":{"mem_util":18.526094,"cpu_util":87.67309}}
{"hostname":"NODE-142","total":8753,"succ_cnt":4227,"succ_rate":0.48292014,"status":0,"timestamp":"2020-01-07T21:29:31.436","tags":{"mem_util":87.426476,"cpu_util":12.49879}}
{"hostname":"NODE-198","total":2261,"succ_cnt":4251,"succ_rate":1.8801415,"status":0,"timestamp":"2020-01-07T21:29:34.444","tags":{"mem_util":5.665392,"cpu_util":69.29729}}
{"hostname":"NODE-46","total":2846,"succ_cnt":6600,"succ_rate":2.3190444,"status":0,"timestamp":"2020-01-07T21:29:37.451","tags":{"mem_util":66.49958,"cpu_util":18.604118}}
{"hostname":"NODE-47","total":5110,"succ_cnt":9650,"succ_rate":1.888454,"status":1,"timestamp":"2020-01-07T21:29:40.459","tags":{"mem_util":92.13784,"cpu_util":60.55072}}
Processed a total of 195 messages
```



通过命令，我们可以看到kafka中已经有新写进去的数据，那么我们在Spring Boot中又该如何对数据进行消费呢？



## 消费消息

生产数据很简单，消费数据也不难，主要使用Spring Boot提供的注解`@KafkaListener`，这时候我们需要自定义一个consumer类，刚刚我们是往`test-producer`这个topic里面写，现在通过程序消费这个topic

`@KafkaListener` 常用的参数

- topic 设置消费的topic
- groupId 指定消费组

### MetricConsumer

```java

import com.iogogogo.common.util.JsonParse;
import com.iogogogo.kafka.pojo.Metric;
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.Optional;

/**
 * Created by tao.zeng on 2020-01-07.
 */
@Slf4j
@Component
public class MetricConsumer {

    @KafkaListener(topics = "test-producer")
    public void event(ConsumerRecord<String, String> record) {
        Optional<String> kafkaMessage = Optional.ofNullable(record.value());
        kafkaMessage.ifPresent(x -> {
            Metric metric = JsonParse.parse(x, Metric.class);
            log.info("消费kafka中的数据:{}", metric);
        });
    }
}

```

消费到的数据结果

```verilog
2020-01-07 21:38:10.646  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-128, total=7849, succCnt=9252, succRate=1.1787488, status=1, timestamp=2020-01-07T21:38:10.639, tags={mem_util=83.20216, cpu_util=2.36848})
2020-01-07 21:38:13.651  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-199, total=3799, succCnt=595, succRate=0.15662016, status=0, timestamp=2020-01-07T21:38:13.647, tags={mem_util=37.34904, cpu_util=56.30006})
2020-01-07 21:38:16.662  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-82, total=2797, succCnt=3004, succRate=1.0740079, status=1, timestamp=2020-01-07T21:38:16.654, tags={mem_util=87.53287, cpu_util=53.029602})
2020-01-07 21:38:19.671  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-102, total=3232, succCnt=3041, succRate=0.9409035, status=0, timestamp=2020-01-07T21:38:19.664, tags={mem_util=87.7293, cpu_util=51.062298})
2020-01-07 21:38:22.679  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-173, total=904, succCnt=759, succRate=0.83960176, status=0, timestamp=2020-01-07T21:38:22.672, tags={mem_util=3.923422, cpu_util=36.679466})
2020-01-07 21:38:25.685  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-4, total=2250, succCnt=8072, succRate=3.5875556, status=1, timestamp=2020-01-07T21:38:25.681, tags={mem_util=71.37522, cpu_util=2.6286244})
2020-01-07 21:38:28.696  INFO 42501 --- [ntainer#0-0-C-1] c.i.kafka.consumer.MetricConsumer        : 消费kafka中的数据:Metric(hostname=NODE-87, total=2798, succCnt=3955, succRate=1.4135096, status=0, timestamp=2020-01-07T21:38:28.689, tags={mem_util=98.27863, cpu_util=23.622477})
```



但是大家发现了一点，就是我们的`topic`都是写死在程序里面的，不能动态传递读取配置文件，这样的肯定是不可以的，那么如何动态配置又是一个新的问题，有人可能会说这里使用`@Value`进行注入，但是实际这样是编译不过去的，因为注解属性的值必须是一个constant，解决方法就是使用`SPEL`表达式

Issues地址：[动态配置多个topic](https://github.com/spring-projects/spring-kafka/issues/361)



修改以后的代码

```java
package com.iogogogo.kafka.consumer;

import com.iogogogo.common.util.JsonParse;
import com.iogogogo.kafka.pojo.Metric;
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.Optional;

/**
 * Created by tao.zeng on 2020-01-07.
 * <p>
 * 动态配置多个topic
 * * https://github.com/spring-projects/spring-kafka/issues/361
 */
@Slf4j
@Component
public class MetricConsumer {


    /**
     * 注意这里的topic配置
     * 当然groupId也可以使用SPEL表达书进行配置，这里就不在赘述
     *
     * @param record
     */
    @KafkaListener(topics = "#{'${kafka.producer.topic}'}", groupId = "metric-group")
    public void event(ConsumerRecord<String, String> record) {
        Optional<String> kafkaMessage = Optional.ofNullable(record.value());
        kafkaMessage.ifPresent(x -> {
            Metric metric = JsonParse.parse(x, Metric.class);
            log.info("消费kafka中的数据:{}", metric);
        });
    }
}
```



修改完成，重启进程，查看依旧可以正常消费到数据，并且topic是动态配置的，如果需要配置多个使用spilt进行分割即可





## 总结

本文主要介绍了Spring Boot与kafka 的整合，日常开发中除了kafka还有其他的各种消息中间件，整合方式大同小异，毕竟Spring Boot已经帮我们封装的很好了，唯一需要注意的就是注解属性的动态注入，这里需要使用SPEL表达式。