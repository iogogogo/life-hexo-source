---
title: vertica开启ROS
date: 2020-03-12 15:07:01
tags: Vertica
categories: Vertica
---




vertica默认批量插入是关闭的，需要手动设置`DataSourceProperties`开启，详细参考`JDBC Connection Properties`，文档中该参数介绍如下：

- DirectBatchInsert

> Determines whether a batch insert stored data directly into [ROS](../../Glossary/ROSReadOptimizedStore.htm) (true) or using [AUTO](javascript:void(0);) mode (false).
>
> When you load data using AUTO mode, Vertica inserts the data first into the [WOS](../../Glossary/WOSWriteOptimizedStore.htm). If the WOS is full, Vertica inserts the data directly into [ROS](../../Glossary/ROSReadOptimizedStore.htm). For details about load options, see [Choosing a Load Method](../../AdministratorsGuide/BulkLoadCOPY/ChoosingALoadMethod.htm).
>
> **Set After Connection:**` VerticaConnection.setProperty()`
>
> **Default Value:** false



在jdbc方式和spring中集成数据源方式略有不同，下面分开介绍



### `JDBC`方式

jdbc方式配置比较灵活，可以在`DriverManager.getConnection`获取连接设置属性

也可以获取到`connection`以后设置

- 获取连接时

```java
Properties myProp = new Properties();
myProp.put("user", "ExampleUser");
myProp.put("password", "password123");
// Enable directBatchInsert for this connection
myProp.put("DirectBatchInsert", "true");
Connection conn;
try {
    conn = DriverManager.getConnection(
         "jdbc:vertica://VerticaHost:5433/ExampleDB", myProp);
. . .
```

- 获取连接后

```java
((VerticaConnection)conn).setProperty("DirectBatchInsert", true);
```





### 使用`HikariDataSource`连接池

当我们使用spring框架开发时，一般会使用数据库连接池对象，spring boot中默认的连接池是`HikariCP`，下面介绍``HikariCP`如何配置该参数，思路可以参考使用jdbc时获取连接以后的方式。

但是又因为spring中的bean是基于proxy进行创建的，所有我们获取到的连接对象不再是`VerticaConnection`，而是`HikariProxyConnection`，但是该对象并没有`setProperty()`方法，所以**不能使用强制类型转换并且设置该属性**。

```java
Connection connection = dataSource.getConnection();

boolean flag1 = connection instanceof com.vertica.jdbc.VerticaConnection; // false
boolean flag2 = connection instanceof com.zaxxer.hikari.pool.HikariProxyConnection; //true
```



**正确做法如下：**

获取到当前连接的`HikariDataSource`对象，然后给该对象设置`DataSourceProperties`即可开启`ROS`，并且可以使用连接池管理`jdbc`连接




```java
public DataSource config(DataSource dataSource, HikariConfig config, boolean isVertica) {
  HikariDataSource hikariDataSource = null;
  if (dataSource instanceof HikariDataSource) {
    // 连接池配置
    hikariDataSource = (HikariDataSource) dataSource;

    if (isVertica) {
      Properties properties = new Properties();
      // Loading Batches Directly into ROS Enable directBatchInsert for this connection
      properties.put("DirectBatchInsert", "true");
      hikariDataSource.setDataSourceProperties(properties);
    }

    hikariDataSource.setPoolName(config.getPoolName());
    hikariDataSource.setAutoCommit(config.isAutoCommit());
    hikariDataSource.setConnectionTestQuery(config.getConnectionTestQuery());
    hikariDataSource.setIdleTimeout(config.getIdleTimeout());
    hikariDataSource.setConnectionTimeout(config.getConnectionTimeout());
    hikariDataSource.setMaximumPoolSize(config.getMaximumPoolSize());
    hikariDataSource.setMaxLifetime(config.getMaxLifetime());
    hikariDataSource.setMinimumIdle(config.getMinimumIdle());
  }
  return hikariDataSource == null ? dataSource : hikariDataSource;
}
```