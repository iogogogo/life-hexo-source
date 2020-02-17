---
title: Spring事务管理
date: 2020-02-16 14:42:08
tags: 
	- spring
	- transaction
categories: Spring
---



## 什么是事务

事务是逻辑上的一组操作，要么都执行，要么都不执行。

### 原子性

> 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

### 一致性

> 执行事务前后，数据保持一致；

### 持久性

> 并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；

### 隔离性

> 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。



## Spring 事务接口介绍

- PlatformTransactionManager 事务管理器
- TransactionDefinition 事务定义形象（隔离、传播、超时、只读）
- TransactionStatus 事务具体的运行状态



### PlatformTransactionManager

**Spring并不直接管理事务，而是提供了多种事务管理器** ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring事务管理器的接口是： **org.springframework.transaction.PlatformTransactionManager** ，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

![PlatformTransactionManager](/images/spring/spring-platformTransactionManager.png)



| 事务                                                         | 说明                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| org.springframework.jdbc.datasource.DataSourceTransactionManager | 使用Spring Jdbc或者MyBatis进行持久化数据时使用 |
| org.springframework.orm.hibernate5.HibernateTransactionManager | 使用Hibernate5.0版本进行持久化数据使用         |
| org.springframework.orm.jpa.JpaTransactionManager            | 使用Jpa持久化使用                              |
| org.springframework.kafka.transaction.KafkaTransactionManager | 使用Kafka事务时使用                            |



### TransactionDefinition

事务管理器接口 **PlatformTransactionManager** 通过 **getTransaction(TransactionDefinition definition)** 方法来得到一个事务，这个方法里面的参数是 **TransactionDefinition类** ，这个类就定义了一些基本的事务属性。

![TransactionDefinition](/images/spring/spring-transactionDefinition.jpeg)



#### 隔离级别

在不考虑隔离性的情况下，会引发如下问题

- 脏度 

> 一个事务读取了另一个事务改写但为提交的数据，如果这些数据被回滚，则独到的数据时无效的

- 不可重复度

> 在同一事务中，多次读取同一数据返回的结果不一致

- 幻读

> 一个事务读取了几行记录后，另一个事务插入一些记录，幻读就发生了
>
> 在后来的查询中，第一个事务就会发现有些原来没有的记录



| 隔离级别                   | 导致的问题                                                   |
| -------------------------- | ------------------------------------------------------------ |
| **ISOLATION_DEFAULT**      | 使用数据库默认的隔离级别（spring默认）                       |
| ISOLATION_READ_UNCOMMITTED | 允许读取还未提交的改变了的数据，可能导致脏、幻、不可重复读   |
| ISOLATION_READ_COMMITTED   | 允许在并发事务已经提交后读取。可防止脏读，但幻、不可重复读任可发生 |
| ISOLATION_REPEATABLE_READ  | 对相同字段的多次读取是一致的，除非数据被事务本身改变。可防止脏、不可重复读，但幻读仍可能发生 |
| ISOLATION_SERIALIZABLE     | 完全服从ACID的隔离级别，事务只能一个一个执行，避免了脏读、不可重复读、幻读。执行效率慢，使用时慎重 |



注：MySQL使用的是`REPEATABLE_READ`；Oracle使用的是`READ_COMMITTED`



#### 传播行为

解决业务层方法之间的相互调用的问题，事务传递方式。

| 常量名称                     | 常量解释                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| **PROPAGATION_REQUIRED**     | 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是 Spring 默认的事务的传播。 |
| PROPAGATION_SUPPORTS         | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY        | 支持当前事务，如果当前没有事务，就抛出异常。                 |
| **PROPAGATION_REQUIRES_NEW** | 如果事务存在，挂起当前事务，创建一个新的事务                 |
| PROPAGATION_NOT_SUPPORTED    | 以非事务方式运行，如果当前存在事务，就把当前事务挂起。       |
| PROPAGATION_NEVER            | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| **PROPAGATION_NESTED**       | 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。 |



### TransactionStatus

`TransactionStatus`接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.

`PlatformTransactionManager.getTransaction(…) `方法返回一个 `TransactionStatus` 对象。返回的`TransactionStatus` 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。

![TransactionStatus-diagram](/images/spring/spring-transactionStatus-diagram.jpeg)

![TransactionStatus](/images/spring/spring-transactionStatus.jpeg)

![TransactionExecution](/images/spring/spring-transactionExecution.jpeg)



## Spring中的事务管理

### 编程式的事务管理

使用`TransactionTemplate`手动管理事务，需要修改代码，侵入性相对较大，一般不使用

### 声明式的事务管理

#### TransactionProxyFactoryBean

需要为每个进行事务管理的类，配置一个`TransactionProxyFactoryBean`进行增强处理

#### 基于AspectJ的AOP方式（xml）

早期纯粹的`spring mvc`配置形式，一旦配置好以后不需要修改任何代码

#### 基于`@Transactional`注解方式

现在`spring boot`常用方式，只需要在需要事务处理的类或者方法添加`@Transactional`。下面使用注解方式简单做一个转账的事务模拟。



## 使用转账模拟事务管理

使用转账模拟一下事务处理，这里演示两种操作数据库的方式，用于对比查看



### 项目结构

```shell
➜  example-transaction git:(master) ✗ tree
.
├── example-transaction.iml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── iogogogo
    │   │           └── transfer
    │   │               ├── TransferApplication.java
    │   │               ├── entity
    │   │               │   └── TransferAccount.java
    │   │               ├── jpa
    │   │               │   └── TransferRepository.java
    │   │               ├── mapper
    │   │               │   └── TransferMapper.java
    │   │               └── service
    │   │                   ├── TransferService.java
    │   │                   └── impl
    │   │                       └── TransferServiceImpl.java
    │   └── resources
    │       ├── application.yml
    │       └── transfer.sql
    └── test
        └── java
            └── com
                └── iogogogo
                    └── transfer
                        └── TransferApplicationTests.java
```



### 新建数据库表

```sql
drop table if exists transfer_account;

create table transfer_account
(
    id    int primary key auto_increment,
    name  varchar(255) unique not null,
    money float default null
);

insert into transfer_account (id, name, money)
values (1, 'jack.zhang', 1000);
insert into transfer_account (id, name, money)
values (2, 'kevin.yu', 1000);

select *
from transfer_account;
```



### pom配置

```xml
<dependencies>
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1.tmp</version>
  </dependency>
</dependencies>
```

### application.yml

```yaml
server:
  port: 8084
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/transfer?characterEncoding=utf8&useSSL=false&allowMultiQueries=true
    username: root
    password: MySQL@123
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari: # http://blog.didispace.com/Springboot-2-0-HikariCP-default-reason/
      minimum-idle: 5
      maximum-pool-size: 20
      auto-commit: true
      idle-timeout: 30000
      pool-name: TransferHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
  jpa:
    database-platform: org.hibernate.dialect.MySQL57Dialect
    open-in-view: false
    show-sql: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        use_sql_comments: true
        format_sql: true
logging:
  level:
    com.iogogogo.transfer.mapper: debug

```



### Jpa操作数据库

```java
package com.iogogogo.transfer.jpa;

import com.iogogogo.transfer.entity.TransferAccount;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

/**
 * Created by tao.zeng on 2020/2/16.
 */
@Repository
public interface TransferRepository extends JpaRepository<TransferAccount, Long> {

    /**
     * @param in    转入人
     * @param money 金额
     */
    @Modifying
    @Query(value = "update transfer_account set money = money + ?2 where name = ?1", nativeQuery = true)
    void inMoney(String in, float money);

    /**
     * @param out   转出人
     * @param money 金额
     */
    @Modifying
    @Query(value = "update transfer_account set money = money - :money where name = :name", nativeQuery = true)
    void outMoney(@Param("name") String out, @Param("money") float money);
}

```



#### 业务层模拟一个转账操作

```java
package com.iogogogo.transfer.service.impl;

import com.iogogogo.transfer.jpa.TransferRepository;
import com.iogogogo.transfer.mapper.TransferMapper;
import com.iogogogo.transfer.service.TransferService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.support.TransactionTemplate;

/**
 * Created by tao.zeng on 2020/2/16.
 */
@Service
public class TransferServiceImpl implements TransferService {

    @Autowired
    private TransferRepository transferRepository;

    @Autowired
    private TransferMapper transferMapper;

    @Autowired
    private TransactionTemplate transactionTemplate;

    @Override
    public void transfer(String in, String out, float money) {
        // 这里使用的是jpa操作数据库
        transferRepository.outMoney(out, money);
        transferRepository.inMoney(in, money);
    }
}

```



#### 新建测试类

```java
package com.iogogogo.transfer;

import com.iogogogo.transfer.service.TransferService;
import lombok.extern.slf4j.Slf4j;
import org.junit.Ignore;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * Created by tao.zeng on 2020/2/16.
 */
@Slf4j
@Ignore
@org.junit.runner.RunWith(SpringRunner.class)
@org.springframework.boot.test.context.SpringBootTest(classes = TransferApplication.class)
public class TransferApplicationTests {

    @Autowired
    private TransferService transferService;

    @Test
    public void test1() {
        transferService.transfer("jack.zhang", "kevin.yu", 200);
    }
}

```

但我们运行时，会抛出如下异常：

```verilog

org.springframework.dao.InvalidDataAccessApiUsageException: Executing an update/delete query; nested exception is javax.persistence.TransactionRequiredException: Executing an update/delete query

	at org.springframework.orm.jpa.EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible(EntityManagerFactoryUtils.java:403)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:257)
	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.translateExceptionIfPossible(AbstractEntityManagerFactoryBean.java:528)
	at org.springframework.dao.support.ChainedPersistenceExceptionTranslator.translateExceptionIfPossible(ChainedPersistenceExceptionTranslator.java:61)
	at org.springframework.dao.support.DataAccessUtils.translateIfNecessary(DataAccessUtils.java:242)
	at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:153)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.data.jpa.repository.support.CrudMethodMetadataPostProcessor$CrudMethodMetadataPopulatingMethodInterceptor.invoke(CrudMethodMetadataPostProcessor.java:149)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:93)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212)
	at com.sun.proxy.$Proxy99.outMoney(Unknown Source)
	at com.iogogogo.transfer.service.impl.TransferServiceImpl.transfer(TransferServiceImpl.java:28)
	at com.iogogogo.transfer.TransferApplicationTests.test1(TransferApplicationTests.java:24)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.springframework.test.context.junit4.statements.RunBeforeTestExecutionCallbacks.evaluate(RunBeforeTestExecutionCallbacks.java:74)
	at org.springframework.test.context.junit4.statements.RunAfterTestExecutionCallbacks.evaluate(RunAfterTestExecutionCallbacks.java:84)
	at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)
	at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)
	at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:251)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:97)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:190)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)
Caused by: javax.persistence.TransactionRequiredException: Executing an update/delete query
	at org.hibernate.internal.AbstractSharedSessionContract.checkTransactionNeededForUpdateOperation(AbstractSharedSessionContract.java:409)
	at org.hibernate.query.internal.AbstractProducedQuery.executeUpdate(AbstractProducedQuery.java:1601)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.orm.jpa.SharedEntityManagerCreator$DeferredQueryInvocationHandler.invoke(SharedEntityManagerCreator.java:409)
	at com.sun.proxy.$Proxy115.executeUpdate(Unknown Source)
	at org.springframework.data.jpa.repository.query.JpaQueryExecution$ModifyingExecution.doExecute(JpaQueryExecution.java:238)
	at org.springframework.data.jpa.repository.query.JpaQueryExecution.execute(JpaQueryExecution.java:88)
	at org.springframework.data.jpa.repository.query.AbstractJpaQuery.doExecute(AbstractJpaQuery.java:154)
	at org.springframework.data.jpa.repository.query.AbstractJpaQuery.execute(AbstractJpaQuery.java:142)
	at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.doInvoke(RepositoryFactorySupport.java:618)
	at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.invoke(RepositoryFactorySupport.java:605)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor.invoke(DefaultMethodInvokingMethodInterceptor.java:80)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:366)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:99)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.dao.support.PersistenceExceptionTranslationInterceptor.invoke(PersistenceExceptionTranslationInterceptor.java:139)
	... 39 more


```

那是因为使用jpa的时候，涉及到`@Modifying`就表示是增删改操作，那么就必须添加`@Transactional`，当然如果是单纯的查询就不需要了（spring考虑的挺全面）。当加上`@Transactional`就可以正常运行了。

我们可以在代码上手动给他添加一个异常，再来看结果。这里我们加了一个`int i = 1 / 0;`会产生一个除以0的异常，因为有`@Transactional`的存在，整个执行是不能成功的。

```java
@Override
@Transactional
public void transfer(String in, String out, float money) {
  // 这里使用的是jpa操作数据库
  transferRepository.outMoney(out, money);
  int i = 1 / 0;
  transferRepository.inMoney(in, money);
}
```





### MyBatis-Plus操作数据库

```java
package com.iogogogo.transfer.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.iogogogo.transfer.entity.TransferAccount;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Update;
import org.springframework.stereotype.Repository;

/**
 * Created by tao.zeng on 2020/2/16.
 */
@Repository
public interface TransferMapper extends BaseMapper<TransferAccount> {

    /**
     * @param in    转入人
     * @param money 金额
     */
    @Update("update transfer_account set money = money + #{money} where name = #{name}")
    void inMoney(@Param("name") String in, @Param("money") float money);

    /**
     * @param out   转出人
     * @param money 金额
     */
    @Update("update transfer_account set money = money - #{money} where name = #{name}")
    void outMoney(@Param("name") String out, @Param("money") float money);
}
```



#### 业务层模拟一个转账操作

```java
package com.iogogogo.transfer.service.impl;

import com.iogogogo.transfer.jpa.TransferRepository;
import com.iogogogo.transfer.mapper.TransferMapper;
import com.iogogogo.transfer.service.TransferService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionTemplate;

/**
 * Created by tao.zeng on 2020/2/16.
 */
@Service
public class TransferServiceImpl implements TransferService {

    @Autowired
    private TransferRepository transferRepository;

    @Autowired
    private TransferMapper transferMapper;

    @Autowired
    private TransactionTemplate transactionTemplate;

    @Override
    public void transfer(String in, String out, float money) {
        // 这里使用的是MyBatis-Plus操作数据库
        transferMapper.outMoney(out, money);
      	// int i = 1 / 0;
        transferMapper.inMoney(in, money);
    }
}

```

在没有异常情况下，就算不加` @Transactional`是可以直接操作成功的，但是如果 `int i = 1 / 0;`存在，`outMoney`方法还是可以正常执行的，这样就会造成钱被扣了但是对方却没有收到的情况，所以这时候就需要添加` @Transactional`让他进行事务处理，这里mybatis就没有jpa处理的好了。



##  @Transactional 介绍


| **参数**                   | **描述**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| **readOnly**               | 该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。例如：@Transactional(readOnly=true) |
| **rollbackFor**            | 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class}) |
| **rollbackForClassName**   | 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：指定单一异常类名称：@Transactional(rollbackForClassName="RuntimeException")指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","Exception"}) |
| **noRollbackFor**          | 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。例如：指定单一异常类：@Transactional(noRollbackFor=RuntimeException.class)指定多个异常类：@Transactional(noRollbackFor={RuntimeException.class, Exception.class}) |
| **noRollbackForClassName** | 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。例如：指定单一异常类名称：@Transactional(noRollbackForClassName="RuntimeException")指定多个异常类名称：@Transactional(noRollbackForClassName={"RuntimeException","Exception"}) |
| **propagation**            | 该属性用于设置事务的传播行为，具体取值可参考表6-7。例如：@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true) |
| **isolation**              | 该属性用于设置底层数据库的事务隔离级别，事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置 |
| **timeout**                | 该属性用于设置事务的超时秒数，默认值为-1表示永不超时         |

以上参考：[`spring`事务注解](https://www.cnblogs.com/caoyc/p/5632963.html)



## 总结

以上介绍了一下spring中的事务接口和事务的传播行为以及隔离级别，也通过转账的例子演示了有事务和没有事务的区别，再有了`spring boot`以后，基本都是使用注解的形式进行事务处理。





源码地址：https://github.com/iogogogo/life-example/tree/master/example-transaction