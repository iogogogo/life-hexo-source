---
title: Spring Boot 整合MyBatis-Plus使用多数据源
date: 2019-03-19 22:34:14
tags: Spring Boot
categories: Spring Boot
---

项目中使用到了MySQL数据库存储配置数据，Vertica中存储指标数据，这样就有两个基于jdbc的数据源，所以需要做到动态配置与切换，并且项目采用了[mybatis-plus](https://mp.baomidou.com/)作为orm框架，所以使用mybatis-plus配置多数据源，并且配置hikari连接池，这也是Spring Boot-2.x自带的连接池，这里提供一个配置思路与方案，仅供参考。通过查看mybatis-plus的源码发现，该框架目前连接Vertica时会提示一个警告⚠️ 表示不支持该数据库，实际使用时可以直接使用mybatis执行sql的功能即可。

```
2019-03-19 17:36:20.877  WARN 14103 --- [  restartedMain] c.b.m.extension.toolkit.JdbcUtils        : The jdbcUrl is jdbc:vertica://192.168.21.188:5433/vertica20190122001, Mybatis Plus Cannot Read Database type or The Database's Not Supported!
```

关于MySQL和Vertica的建库建表这边就不放了，直接贴上核心代码。

## pom 配置

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>${mybatisplus.spring.version}</version>
</dependency>
<!-- jdbc driver 这是是官网下载，自己传到内网服务器的jdbc驱动 -->
<dependency>
  <groupId>com.iogogogo.vertica</groupId>
  <artifactId>vertica-jdbc</artifactId>
  <version>9.2.0</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
</dependencies>
```



## 配置文件

```yaml
spring:
  application:
    name: example-dynamic-datasource
  datasource:
    mysql:
      driver-class-name: com.mysql.cj.jdbc.Driver
      hikari:
        auto-commit: true
        connection-test-query: SELECT 1
        connection-timeout: 30000
        idle-timeout: 30000
        max-lifetime: 1800000
        maximum-pool-size: 15
        minimum-idle: 5
        pool-name: MySQLHikariCP
      password: Root@123
      type: com.zaxxer.hikari.HikariDataSource
      jdbc-url: jdbc:mysql://192.168.21.111:3306/life-test?characterEncoding=utf8&useSSL=false
      username: root
    vertica:
      driver-class-name: com.vertica.jdbc.Driver
      hikari:
        auto-commit: true
        connection-test-query: SELECT 1
        connection-timeout: 30000
        idle-timeout: 30000
        max-lifetime: 1800000
        maximum-pool-size: 15
        minimum-idle: 5
        pool-name: VerticaHikariCP
      password: 123456
      type: com.zaxxer.hikari.HikariDataSource
      jdbc-url: jdbc:vertica://192.168.21.188:5433/vertica20190122001
      username: dbadmin

logging:
  home: ${user.dir}/logs
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
  type-aliases-package: com.iogogogo.entity

```

## HikariCP 配置

```java
package com.iogogogo.datasource.config;

import lombok.Data;

/**
 * Created by tao.zeng on 2019-03-20.
 */
@Data
public class HikariConfig {

    private String poolName;

    private boolean autoCommit;

    private long connectionTimeout;

    private long idleTimeout;

    private long maxLifetime;

    private int maximumPoolSize;

    private int minimumIdle;

    private String connectionTestQuery;
}
```



## MySQL数据源配置

### 连接池配置

```java
package com.iogogogo.datasource.config;

import lombok.Data;
import lombok.EqualsAndHashCode;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

/**
 * Created by tao.zeng on 2019-03-20.
 */
@Data
@Configuration
@EqualsAndHashCode(callSuper = true)
@ConfigurationProperties(prefix = "spring.datasource.mysql.hikari")
public class HikariMySQLConfig extends HikariConfig {
}
```

### DataSource配置

```java
package com.iogogogo.datasource;

import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
import com.iogogogo.datasource.configure.HikariMySQLConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

/**
 * Created by tao.zeng on 2019-03-19.
 */
@Configuration
@MapperScan(basePackages = "com.iogogogo.mapper.mysql", sqlSessionTemplateRef = "mysqlSqlSessionTemplate")
public class MySQLDataSourceConfigure {

    private HikariMySQLConfig mysqlConfig;

    public MySQLDataSourceConfigure(HikariMySQLConfig mysqlConfig) {
        this.mysqlConfig = mysqlConfig;
    }

    @Bean(name = "mysqlDataSource")
    @ConfigurationProperties("spring.datasource.mysql")
    public DataSource mysql() {
        DataSource dataSource = DataSourceBuilder.create().build();
        HikariDataSource hikariDataSource = null;
        if (dataSource instanceof HikariDataSource) {
            // 连接池配置
            hikariDataSource = (HikariDataSource) dataSource;
            hikariDataSource.setPoolName(mysqlConfig.getPoolName());
            hikariDataSource.setAutoCommit(mysqlConfig.isAutoCommit());
            hikariDataSource.setConnectionTestQuery(mysqlConfig.getConnectionTestQuery());
            hikariDataSource.setIdleTimeout(mysqlConfig.getIdleTimeout());
            hikariDataSource.setConnectionTimeout(mysqlConfig.getConnectionTimeout());
            hikariDataSource.setMaximumPoolSize(mysqlConfig.getMaximumPoolSize());
            hikariDataSource.setMaxLifetime(mysqlConfig.getMaxLifetime());
            hikariDataSource.setMinimumIdle(mysqlConfig.getMinimumIdle());
        }
        return hikariDataSource == null ? dataSource : hikariDataSource;
    }

    @Bean(name = "mysqlSqlSessionFactory")
    public SqlSessionFactory mysqlSqlSessionFactory(@Qualifier("mysqlDataSource") DataSource dataSource) throws Exception {
        // MyBatis-Plus使用MybatisSqlSessionFactoryBean  MyBatis直接使用SqlSessionFactoryBean
        MybatisSqlSessionFactoryBean bean = new MybatisSqlSessionFactoryBean();
        // 给MyBatis-Plus注入数据源
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "mysqlTransactionManager")
    public DataSourceTransactionManager mysqlTransactionManager(@Qualifier("mysqlDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "mysqlSqlSessionTemplate")
    public SqlSessionTemplate mysqlSqlSessionTemplate(@Qualifier("mysqlSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```



## Vertica数据源配置

### 连接池配置

```java
package com.iogogogo.datasource.config;

import lombok.Data;
import lombok.EqualsAndHashCode;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

/**
 * Created by tao.zeng on 2019-03-20.
 */
@Data
@Configuration
@EqualsAndHashCode(callSuper = true)
@ConfigurationProperties(prefix = "spring.datasource.vertica.hikari")
public class HikariVerticaConfig extends HikariConfig {
}
```

### DataSource配置

```java
package com.iogogogo.datasource;

import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
import com.iogogogo.datasource.configure.HikariVerticaConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

/**
 * Created by tao.zeng on 2019-03-19.
 */
@Configuration
@MapperScan(basePackages = "com.iogogogo.mapper.vertica", sqlSessionTemplateRef = "verticaSqlSessionTemplate")
public class VerticaDataSourceConfigure {

    private HikariVerticaConfig verticaConfig;

    public VerticaDataSourceConfigure(HikariVerticaConfig verticaConfig) {
        this.verticaConfig = verticaConfig;
    }

    @Bean(name = "verticaDataSource")
    @ConfigurationProperties("spring.datasource.vertica")
    public DataSource vertica() {
        DataSource dataSource = DataSourceBuilder.create().build();
        HikariDataSource hikariDataSource = null;
        if (dataSource instanceof HikariDataSource) {
            // 连接池配置
            hikariDataSource = (HikariDataSource) dataSource;
            hikariDataSource.setPoolName(verticaConfig.getPoolName());
            hikariDataSource.setAutoCommit(verticaConfig.isAutoCommit());
            hikariDataSource.setConnectionTestQuery(verticaConfig.getConnectionTestQuery());
            hikariDataSource.setIdleTimeout(verticaConfig.getIdleTimeout());
            hikariDataSource.setConnectionTimeout(verticaConfig.getConnectionTimeout());
            hikariDataSource.setMaximumPoolSize(verticaConfig.getMaximumPoolSize());
            hikariDataSource.setMaxLifetime(verticaConfig.getMaxLifetime());
            hikariDataSource.setMinimumIdle(verticaConfig.getMinimumIdle());
        }
        return hikariDataSource == null ? dataSource : hikariDataSource;
    }

    @Bean(name = "verticaSqlSessionFactory")
    public SqlSessionFactory verticaSqlSessionFactory(@Qualifier("verticaDataSource") DataSource dataSource) throws Exception {
        // MyBatis-Plus使用MybatisSqlSessionFactoryBean  MyBatis直接使用SqlSessionFactoryBean
        MybatisSqlSessionFactoryBean bean = new MybatisSqlSessionFactoryBean();
        // 给MyBatis-Plus注入数据源
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "verticaTransactionManager")
    public DataSourceTransactionManager verticaTransactionManager(@Qualifier("verticaDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "verticaSqlSessionTemplate")
    public SqlSessionTemplate verticaSqlSessionTemplate(@Qualifier("verticaSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

## Mapper

### MySQL

```java
package com.iogogogo.mapper.mysql;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.iogogogo.entity.SysUser;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * Created by tao.zeng on 2019-03-19.
 */
@Repository
public interface SysUserMapper extends BaseMapper<SysUser> {

    @Insert("insert into sys_user values(#{x.id},#{x.name},#{x.birthday})")
    boolean save(@Param("x") SysUser user);

    @Select("select * from sys_user")
    List<SysUser> list();
}

```

### Vertica

```java
package com.iogogogo.mapper.vertica;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.iogogogo.entity.User;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * Created by tao.zeng on 2019-03-15.
 */
@Repository
public interface UserMapper extends BaseMapper<User> {

    @Select("select * from public.user")
    List<User> list();
}
```

可以发现只要数据源配置成功以后，两者已经没有任何区别了，就可以像正常写代码一样，需要注意的就是不同数据库的mapper需要放在指定的包下面，否则spring容器无法扫描。



## 测试

为了方便，直接就在项目启动以后查询两个数据库的数据就好了

```java
package com.iogogogo;

import com.iogogogo.entity.SysUser;
import com.iogogogo.entity.User;
import com.iogogogo.mapper.mysql.SysUserMapper;
import com.iogogogo.mapper.vertica.UserMapper;
import com.iogogogo.util.IdHelper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.time.LocalDateTime;
import java.util.List;

/**
 * Created by tao.zeng on 2019-03-15.
 */
@Slf4j
@SpringBootApplication
public class DynamicDataSourceApplication implements CommandLineRunner {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private SysUserMapper sysUserMapper;


    public static void main(String[] args) {
        SpringApplication.run(DynamicDataSourceApplication.class, args);
    }

    @Override
    public void run(String... args) {

        int i = userMapper.insert(new User(IdHelper.id(), "小花脸-" + IdHelper.uuid(), "description-" + IdHelper.uuid()));
        // 使用自定义的查询方法
        List<User> list = userMapper.list();
        log.info("insert result:{} list.size:{}", i, list.size());


        boolean b = sysUserMapper.save(new SysUser(IdHelper.id(), "小花脸-" + IdHelper.uuid(), LocalDateTime.now()));
        // 使用MyBatis-Plus提供的查询方法
        List<SysUser> users = sysUserMapper.selectList(null);
        log.info("insert result:{} list.size:{}", b, users.size());
        users.forEach(x -> log.info(x.toString()));
    }
}
```



项目启动以后，可以看到控制台初始化了多个数据源的日志

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

2019-03-20 10:33:18.233  INFO 21123 --- [  restartedMain] c.iogogogo.DynamicDataSourceApplication  : Starting DynamicDataSourceApplication on iogogogo.local with PID 21123 (/Users/tao.zeng/share/life-example/example-dynamic-datasource/target/classes started by tao.zeng in /Users/tao.zeng/share/life-example)
2019-03-20 10:33:18.237  INFO 21123 --- [  restartedMain] c.iogogogo.DynamicDataSourceApplication  : No active profile set, falling back to default profiles: default
2019-03-20 10:33:18.360  INFO 21123 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable
2019-03-20 10:33:18.360  INFO 21123 --- [  restartedMain] .e.DevToolsPropertyDefaultsPostProcessor : For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG'
2019-03-20 10:33:20.341  INFO 21123 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-03-20 10:33:20.374  INFO 21123 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-03-20 10:33:20.375  INFO 21123 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.16]
2019-03-20 10:33:20.391  INFO 21123 --- [  restartedMain] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/Users/tao.zeng/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.]
2019-03-20 10:33:20.497  INFO 21123 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-20 10:33:20.497  INFO 21123 --- [  restartedMain] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2136 ms
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.0.6 
2019-03-20 10:33:20.751  INFO 21123 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : VerticaHikariCP - Starting...
2019-03-20 10:33:21.449  INFO 21123 --- [  restartedMain] com.zaxxer.hikari.pool.PoolBase          : VerticaHikariCP - Driver does not support get/set network timeout for connections. (com.vertica.jdbc.VerticaJdbc4ConnectionImpl.getNetworkTimeout()I)
2019-03-20 10:33:21.672  INFO 21123 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : VerticaHikariCP - Start completed.
2019-03-20 10:33:21.683  WARN 21123 --- [  restartedMain] c.b.m.extension.toolkit.JdbcUtils        : The jdbcUrl is jdbc:vertica://192.168.21.188:5433/vertica20190122001, Mybatis Plus Cannot Read Database type or The Database's Not Supported!
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.0.6 
2019-03-20 10:33:21.865  INFO 21123 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : MySQLHikariCP - Starting...
2019-03-20 10:33:23.201  INFO 21123 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : MySQLHikariCP - Start completed.
2019-03-20 10:33:23.516  INFO 21123 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-03-20 10:33:23.804  INFO 21123 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-03-20 10:33:23.887  INFO 21123 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-03-20 10:33:23.892  INFO 21123 --- [  restartedMain] c.iogogogo.DynamicDataSourceApplication  : Started DynamicDataSourceApplication in 6.41 seconds (JVM running for 7.001)
```



以上完整[代码](https://github.com/iogogogo/life-example)