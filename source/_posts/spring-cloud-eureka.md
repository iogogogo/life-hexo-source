---
title: Spring Cloud Eureka 注册中心
date: 2019-03-17 16:41:17
tags: Spring Cloud
---

# 什么是Spring Cloud

Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

下图是一个基本的Spring Cloud组件架构，核心组件有Eureka、Zuul、Ribbon、Fegin、Hystrix等等。接下来将介绍第一个Eureka组件

![图片来源于网络，侵删](/images/spring-cloud-framework.jpg)



# Eureka注册中心



注册中心，管理各种服务功能包括服务的注册、发现、熔断、负载、降级等，比如dubbo admin后台的各种功能。有了服务中心之后，任何一个服务都不能直接去掉用，都需要通过服务中心来调用。通过服务中心来获取服务你不需要关注你调用的项目IP地址，由几台服务器组成，每次直接去服务中心获取可以使用的服务去调用既可。



Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和Service Discovery实现。也是Spring Cloud体系中最重要最核心的组件之一。



官方介绍:

> Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers.
>
> Eureka 是一个基于 REST 的服务，主要在 AWS 云中使用, 定位服务来进行中间层服务器的负载均衡和故障转移。

用一张图简单说明:

![图片来源于网络，侵删](/images/eureka/eureka-architecture-overview.png)

上图简要描述了Eureka的基本架构，由3个角色组成：

1、Eureka Server

- 提供服务注册和发现

2、Service Provider

- 服务提供方
- 将自身服务注册到Eureka，从而使服务消费方能够找到

3、Service Consumer

- 服务消费方
- 从Eureka获取注册服务列表，从而能够消费服务



## Eureka Server

我们可以用Spring  initialize 构建一个基本的maven项目，基于Spring Cloud Greenwich.SR1 然后自己添加相应的依赖

### pom依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <!-- 默认goal。在mvn package之后，再次打包可执行的jar/war，同时保留mvn package生成的jar/war为.origin -->
                        <!-- https://docs.spring.io/spring-boot/docs/current/maven-plugin/repackage-mojo.html -->
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 启动类

```java
package com.iogogogo.eureka;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * 添加启动代码中添加@EnableEurekaServer注解
 * Created by tao.zeng on 2019-03-15.
 */
@Slf4j
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### application.properties 配置

eureka使用8761端口

```properties
spring.application.name=life-cloud-eureka
server.port=8761
eureka.instance.hostname=localhost
# 实例名称显示IP
eureka.instance.prefer-ip-address=true
# 健康检查
eureka.server.enable-self-preservation=false
# 清理间隔
eureka.server.eviction-interval-timer-in-ms=6000
# 表示是否将自己注册到Eureka Server，默认为true。
eureka.client.register-with-eureka=false
# 表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.fetch-registry=false
# eureka服务地址 多个地址可使用 , 分隔。
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```

然后启动项目，访问

```
http://localhost:8761/
```

![img](/images/eureka/eureka-instance.png)



## Eureka 高可用

注册中心这么关键的服务，如果是单点话，遇到故障就是毁灭性的。在生产中我们可能需要三台或者大于三台的注册中心来保证服务的稳定性，在一个分布式系统中，服务注册中心是最重要的基础部分，理应随时处于可以提供服务的状态。为了维持其可用性，使用集群是很好的解决方案。Eureka通过互相注册的方式来实现高可用的部署，所以我们只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。

### 修改hosts文件

hosts文件中添加以下内容`vim /etc/hosts`

```shell
127.0.0.1 peer1  
127.0.0.1 peer2  
127.0.0.1 peer3  
```

### 修改properties文件

- 创建 application-peer1.properties 将serviceUrl指向peer2、peer3

```properties
spring.application.name=life-cloud-eureka
server.port=8761
eureka.instance.hostname=peer1
# 实例名称显示IP
eureka.instance.prefer-ip-address=true
# 健康检查
eureka.server.enable-self-preservation=false
# 清理间隔
eureka.server.eviction-interval-timer-in-ms=6000
# 表示是否将自己注册到Eureka Server，默认为true。
eureka.client.register-with-eureka=false
# 表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.fetch-registry=false
# eureka服务地址 多个地址可使用 , 分隔。
eureka.client.service-url.defaultZone=http://peer2:8762/eureka/,http://peer3:8763/eureka/
```

- 创建 application-peer2.properties 将serviceUrl指向peer1、peer3

```properties
spring.application.name=life-cloud-eureka
server.port=8762
eureka.instance.hostname=peer2
# 实例名称显示IP
eureka.instance.prefer-ip-address=true
# 健康检查
eureka.server.enable-self-preservation=false
# 清理间隔
eureka.server.eviction-interval-timer-in-ms=6000
# 表示是否将自己注册到Eureka Server，默认为true。
eureka.client.register-with-eureka=false
# 表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.fetch-registry=false
# eureka服务地址 多个地址可使用 , 分隔。
eureka.client.service-url.defaultZone=http://peer1:8761/eureka/,http://peer3:8763/eureka/
```

- 创建 application-peer3.properties 将serviceUrl指向peer1、peer2

```properties
spring.application.name=life-cloud-eureka
server.port=8763
eureka.instance.hostname=peer3
# 实例名称显示IP
eureka.instance.prefer-ip-address=true
# 健康检查
eureka.server.enable-self-preservation=false
# 清理间隔
eureka.server.eviction-interval-timer-in-ms=6000
# 表示是否将自己注册到Eureka Server，默认为true。
eureka.client.register-with-eureka=false
# 表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.fetch-registry=false
# eureka服务地址 多个地址可使用 , 分隔。
eureka.client.service-url.defaultZone=http://peer1:8761/eureka/,http://peer2:8762/eureka/
```

### 打包依次启动

```shell
mvn clean package
```

- 依次启动

```shell
java -jar cloud-eureka-0.0.1.jar --spring.profiles.active=peer1
java -jar cloud-eureka-0.0.1.jar --spring.profiles.active=peer2
java -jar cloud-eureka-0.0.1.jar --spring.profiles.active=peer3
```

启动完成后，浏览器输入：http://localhost:8762/ 效果图如下：

![img](/images/eureka/eureka-cluster.png)

可以在peer2中看到了peer1、peer3的相关的副本信息。至此eureka集群也已经完成了。



以上完整代码[github](https://github.com/iogogogo/life-cloud-example)