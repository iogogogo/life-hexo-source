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

![PlatformTransactionManager](/images/spring-platformTransactionManager.jpeg)



| 事务                                                         | 说明                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| org.springframework.jdbc.datasource.DataSourceTransactionManager | 使用Spring Jdbc或者MyBatis进行持久化数据时使用 |
| org.springframework.orm.hibernate5.HibernateTransactionManager | 使用Hibernate5.0版本进行持久化数据使用         |
| org.springframework.orm.jpa.JpaTransactionManager            | 使用Jpa持久化使用                              |
| org.springframework.kafka.transaction.KafkaTransactionManager | 使用Kafka事务时使用                            |



### TransactionDefinition

事务管理器接口 **PlatformTransactionManager** 通过 **getTransaction(TransactionDefinition definition)** 方法来得到一个事务，这个方法里面的参数是 **TransactionDefinition类** ，这个类就定义了一些基本的事务属性。

![TransactionDefinition](/images/spring-transactionDefinition.jpeg)



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

![TransactionStatus-diagram](/images/spring-transactionStatus-diagram.jpeg)

![TransactionStatus](/images/spring-transactionStatus.jpeg)

![TransactionExecution](/images/spring-transactionExecution.jpeg)



## Spring中的事务管理

### 编程式的事务管理

使用`TransactionTemplate`手动管理事务，需要修改代码，侵入性相对较大，一般不使用

### 声明式的事务管理

Spring中的声明式是通过**AOP**实现的，代码侵入性较小，推荐使用



## 使用转账模拟事务管理





