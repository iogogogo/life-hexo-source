---
title: Spring Boot 使用jpa和H2数据库
date: 2020-01-05 17:44:18
tags: Spring Boot
cover: true
categories: Spring Boot
---

日常测试时没有数据库可用时，使用H2不失为一种解决方案，而且H2也经常用在持久层的单元测试，Spring Boot中使用H2也很简单，只要一些配置即可。



## 项目结构与pom.xml

先看看整体的项目结构

```shell
➜  example-h2 git:(master) ✗ tree /Users/tao.zeng/share/life-example/example-h2
/Users/tao.zeng/share/life-example/example-h2
├── example-h2.iml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── h2
    │   │               ├── H2DbApplication.java
    │   │               ├── entity
    │   │               │   └── UserEntity.java
    │   │               └── repository
    │   │                   └── UserRepository.java
    │   └── resources
    │       ├── application.yml
    │       └── db
    │           ├── data.sql
    │           └── schema.sql
    └── test
        └── java
            └── com
                └── iogogogo
                    └── h2
                        └── H2DbApplicationTests.java

15 directories, 9 files
```



这里使用H2数据，为了方便测试也使用jpa进行数据库操作，第一步肯定是需要引入maven依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
```

## 创建数据库脚本与初始化数据脚本

- schema.sql

```sql
create table user_info
(
  id       bigint auto_increment primary key not null,
  name     varchar(255)                      not null,
  birthday datetime                          not null,
  remark   varchar(2048) default null
);

```

- data.sql

```sql
insert into user_info
values (1, '阿牛', '2020-01-05 15:40:00', '哈哈哈');
```



## 配置文件

```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:h2:mem:h2-test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    username: sa
    password:
    platform: h2
    driver-class-name: org.h2.Driver
    hikari: # http://blog.didispace.com/Springboot-2-0-HikariCP-default-reason/
      minimum-idle: 5
      maximum-pool-size: 20
      auto-commit: true
      idle-timeout: 30000
      pool-name: H2HikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    show-sql: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        use_sql_comments: true
        format_sql: true
  h2:
    console:
      # 进行该配置，程序开启时就会启动h2 web console。当然这是默认的，如果你不想在启动程序时启动h2 web console，那么就设置为false。
      enabled: true
      # 进行该配置，你就可以通过 URL/h2-console访问h2 web console。
      path: /h2-console
      settings:
        trace: false
        # 进行该配置后，h2 web console 就可以在远程访问了。否则只能在本机访问。
        web-allow-others: false
```

`application.yml`中，分别有datasource、jpa、h2等配置

### datasouce

其中`url`使用了h2的配置连接字符串，并且使用了`hikari`数据库连接池，详细信息请查看[Springboot 2.0选择HikariCP作为默认数据库连接池的五大理由](http://blog.didispace.com/Springboot-2-0-HikariCP-default-reason/)

### jpa

jpa中配置了数据库方言`database-platform`，显示sql等配置

### h2

`spring.datasource.schema=classpath:db/schema.sql`进行该配置后，每次启动程序，程序都会运行`resources/db/schema.sql`文件，对数据库的结构进行操作。

`spring.datasource.data=classpath:db/data.sql`进行该配置后，每次启动程序，程序都会运行`resources/db/data.sql`文件，对数据库的数据操作。

#### h2 web console配置

`h2 web console`是一个数据库`GUI`管理应用，就和`phpMyAdmin`类似。程序运行时，会自动启动`h2 web console`。当然你也可以进行如下的配置。

- `spring.h2.console.settings.web-allow-others=true`，进行该配置后，`h2 web console`就可以在远程访问了。否则只能在本机访问。
- `spring.h2.console.path=/h2-console`，进行该配置，你就可以通过 `URL/h2-console`访问`h2 web console`。
- `spring.h2.console.enabled=true`，进行该配置，程序开启时就会启动`h2 web console`。当然这是默认的，如果你不想在启动程序时启动`h2 web console`，那么就设置为false。



## 实体类

```java
package com.example.h2.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;
import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * Created by tao.zeng on 2020-01-05.
 */
@Data
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "user_info")
public class UserEntity implements Serializable {

    @Id
    private Long id;

    private String name;

    @Column(name = "birthday")
    private LocalDateTime birthday;

    private String remark;


    public UserEntity(String name, LocalDateTime birthday, String remark) {
        this.name = name;
        this.birthday = birthday;
        this.remark = remark;
    }
}

```

## repository

```java
package com.example.h2.repository;

import com.example.h2.entity.UserEntity;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Created by tao.zeng on 2020-01-05.
 */
public interface UserRepository extends JpaRepository<UserEntity, Long> {
}

```



## 测试

为了方便查看结果，写一个测试类查看结果

```java

import com.example.h2.H2DbApplication;
import com.example.h2.entity.UserEntity;
import com.example.h2.repository.UserRepository;
import lombok.extern.slf4j.Slf4j;
import org.junit.Ignore;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.Optional;
import java.util.UUID;

/**
 * Created by tao.zeng on 2020-01-05.
 */
@Slf4j
@Ignore
@RunWith(SpringRunner.class)
@SpringBootTest(classes = H2DbApplication.class)
public class H2DbApplicationTests {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void test() {
        // 新增数据
        UserEntity entity = userRepository.save(new UserEntity("阿牛" + UUID.randomUUID().toString(), LocalDateTime.now(), "新增测试"));
        log.info("新增结果:{}\n", entity);

        // 修改数据
        entity = userRepository.save(new UserEntity(1L, "哈哈哈哈", LocalDateTime.now(), "修改以后的结果"));
        log.info("按照id修改结果:{}\n", entity);


        // 按照id查询
        Optional<UserEntity> optional = userRepository.findById(1L);
        optional.ifPresent(x -> log.info("按照id查询结果:{}\n", x));

        // 查询所有
        log.info("查询所有");
        Iterable<UserEntity> iterable = userRepository.findAll();
        iterable.forEach(x -> log.info("item:{}", x));


        // 删除数据
        userRepository.deleteById(1L);


        // 删除id=1的数据以后
        System.out.println();
        log.info("删除id=1的数据以后");
        iterable = userRepository.findAll();
        iterable.forEach(x -> log.info("item:{}", x));
    }
}

```

测试结果如下

```shell
2020-01-05 18:07:40.499  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : 新增结果:UserEntity(id=2, name=阿牛33dc0a6d-29cd-4e79-9ae4-ad64b1588e69, birthday=2020-01-05T18:07:40.383, remark=新增测试)

2020-01-05 18:07:40.527  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : 按照id修改结果:UserEntity(id=1, name=哈哈哈哈, birthday=2020-01-05T18:07:40.500, remark=修改以后的结果)

2020-01-05 18:07:40.578  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : 按照id查询结果:UserEntity(id=1, name=哈哈哈哈, birthday=2020-01-05T18:07:40.500, remark=修改以后的结果)

2020-01-05 18:07:40.578  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : 查询所有
2020-01-05 18:07:40.586  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : item:UserEntity(id=1, name=哈哈哈哈, birthday=2020-01-05T18:07:40.500, remark=修改以后的结果)
2020-01-05 18:07:40.586  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : item:UserEntity(id=2, name=阿牛33dc0a6d-29cd-4e79-9ae4-ad64b1588e69, birthday=2020-01-05T18:07:40.383, remark=新增测试)

2020-01-05 18:07:40.602  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : 删除id=1的数据以后
2020-01-05 18:07:40.604  INFO 28214 --- [           main] com.iogogogo.h2.H2DbApplicationTests     : item:UserEntity(id=2, name=阿牛33dc0a6d-29cd-4e79-9ae4-ad64b1588e69, birthday=2020-01-05T18:07:40.383, remark=新增测试)

```





## 完整代码

https://github.com/iogogogo/life-example/tree/master/example-h2