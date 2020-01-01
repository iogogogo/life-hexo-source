---
layout: post
title: Spring Boot 2.x使用mybatis
tags: Spring Boot
categories: Spring Boot
---

orm框架的本质是简化编程中操作数据库的编码，发展到现在基本上就剩两家了，一个是宣称可以不用写一句SQL的hibernate，一个是可以灵活调试动态sql的mybatis,两者各有特点，在企业级系统开发中可以根据需求灵活使用。

# 使用注解版

## pom依赖

这里使用最新版spring boot【2.0.4.RELEASE】

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zz</groupId>
    <artifactId>spring-boot-sample-mybatis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <modules>
        <module>mybatis-xml</module>
        <module>mybatis-annotation</module>
    </modules>
    <packaging>pom</packaging>

    <name>spring-boot-sample-mybatis</name>
    <description>spring-boot-sample-mybatis project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <mybatis-spring.version>1.3.2</mybatis-spring.version>
        <logback.version>1.2.3</logback.version>
    </properties>

    <dependencies>
        <!-- web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- mybatis-spring-boot-starter -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring.version}</version>
        </dependency>
        <!-- MySQL 驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## application.yml文件

```yaml
spring:
  application:
    name: life-spring-sample-mybatis
  datasource:
    url: jdbc:mysql://localhost:10100/life-mybatis?useUnicode=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root

mybatis:
  type-aliases-package: com.zz.entity

server:
  port: 8080
```

## 实体对象

这里为了方便，使用了[lombok](https://projectlombok.org/)，不了解的同学可以自己Google看一下

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee implements Serializable {

    private long id;

    private String name;

    private int age;

    private Date birthday;

    private int deptId;
}
```

## mapper

```java
@Repository
public interface EmployeeMapper {

    /**
     * 使用@Results注解做字段映射
     *
     * @return
     */
    @Select("select * from employee")
    @Results({
            @Result(property = "deptId", column = "dept_id")
    })
    List<Employee> findAll();

    @Select("select * from employee where id=#{id}")
    Employee findById(long id);

    @Update("update employee set name=#{name},birthday=#{birthday},age=#{age},dept_id=#{deptId} where id=#{id}")
    int updateById(Employee employee);

    @Delete("delete from employee where id=#{id}")
    int deleteById(long id);

    @Insert("insert into employee(name,birthday,age,dept_id) values(#{name},#{birthday},#{age},#{deptId})")
    int save(Employee employee);
}
```

## controller通过前端完成增删改查

```java
@Slf4j
@RestController
@RequestMapping("/api/emp")
public class EmployeeController {

    @Autowired
    private EmployeeMapper employeeMapper;

    @RequestMapping("/")
    public List<Employee> findAll() {
        List<Employee> list = employeeMapper.findAll();
        list.forEach(item -> log.info("emp:{}", item));
        return list;
    }

    @RequestMapping("/{id}")
    public Employee findById(@PathVariable("id") long id) {
        Employee employee = employeeMapper.findById(id);
        log.info("emp:{}", employee);
        return employee;
    }

    @RequestMapping(value = "/", method = RequestMethod.POST)
    public int save(@RequestBody Employee employee) {
        return employeeMapper.save(employee);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public int deleteById(@PathVariable("id") long id) {
        return employeeMapper.deleteById(id);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    public int updateById(@PathVariable("id") long id, @RequestBody Employee employee) {
        employee.setId(id);
        return employeeMapper.updateById(employee);
    }
}
```

运行项目以后就可以使用postman工具进行接口请求测试，对应的使用post、get、put等请求，可以自行测试

```http
localhost:8080/api/emp/
```

response

```json
[
    {
        "id": 1,
        "name": "二丫",
        "age": 16,
        "birthday": "2000-01-01T00:00:00.000+0000",
        "deptId": 1
    },
    {
        "id": 2,
        "name": "小强 ",
        "age": 1,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 2
    },
    {
        "id": 3,
        "name": "史塔克 ",
        "age": 11,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 3
    },
    {
        "id": 5,
        "name": "君临城 ",
        "age": 8,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 4
    },
    {
        "id": 6,
        "name": "乌鸦 ",
        "age": 8,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 1
    },
    {
        "id": 7,
        "name": "龙母 ",
        "age": 4,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 4
    },
    {
        "id": 8,
        "name": "提利昂 ",
        "age": 21,
        "birthday": "2018-06-01T08:33:52.000+0000",
        "deptId": 5
    },
    {
        "id": 9,
        "name": "小花",
        "age": 18,
        "birthday": "2000-01-01T00:00:00.000+0000",
        "deptId": 0
    }
]
```



# 使用xml版

前面我们可以看到，使用注解版的mybatis可以一行xml文件都不写，但是这个所有的sql都硬编码在mapper文件中，对后期的维护和复用不是特别方便，所有接下来我们看看使用xml完成增删改查

## application.yml文件

这里和使用注解的区别就是需要指定mapper文件的位置，为了方便我们还有一张dept表，这次使用dept表来进行演示

```yaml
spring:
  application:
    name: life-spring-sample-mybatis
  datasource:
    url: jdbc:mysql://localhost:10100/life-mybatis?useUnicode=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root

mybatis:
  # 设置实体对象的位置
  type-aliases-package: com.zz.entity
  # 设置mapper文件存放的位置，这里在classpath目录下的mybatis文件夹中
  mapper-locations: classpath:mybatis/*.xml

server:
  port: 8082
```

## dept实体对象

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dept implements Serializable {

    private long id;

    private String name;
}
```

## deptMapper对象

可以看到，我们这里只有mapper接口中的方法声明，没有在上面进行注解操作，那么接下来我们就需要编写mapper.xml文件进行数据库操作，在resources/mybatis文件夹下新建DeptMapper.xml文件

```java
@Repository
public interface DeptMapper {

    int save(Dept dept);

    int deleteById(long id);

    int updateById(Dept dept);

    List<Dept> findAll();

    Dept findById(long id);
}
```

## mapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="com.zz.mapper.DeptMapper">

    <sql id="base_column">

    </sql>

    <insert id="save">
        insert into dept(name) values (#name)
    </insert>

    <update id="updateById">
        update dept set name=#{name} where id=#{id}
    </update>


    <delete id="deleteById">
        delete from dept where id=#{id}
    </delete>


    <select id="findAll" resultType="com.zz.entity.Dept">
        select * from dept
    </select>


    <select id="findById" resultType="com.zz.entity.Dept">
        select * from dept where id=#{id}
    </select>
</mapper>
```

## 控制器对象

这里只是为了演示使用xml操作，controller中就不一一写出所有的接口，其实和使用注解完全是一样的

```java
@RestController
@RequestMapping("/api/dept")
public class DeptController {

    @Autowired
    private DeptMapper deptMapper;

    @GetMapping("/")
    public List<Dept> findAll() {
        return deptMapper.findAll();
    }
}
```

这里使用postman工具进行简单的测试，可以看到所有的部门数据全部被查出来了

```http
localhost:8082/api/dept/
```

response

```java
[
    {
        "id": 1,
        "name": "信息技术部"
    },
    {
        "id": 2,
        "name": "人事部"
    },
    {
        "id": 3,
        "name": "PCB事业部"
    },
    {
        "id": 4,
        "name": "无线终端部"
    },
    {
        "id": 5,
        "name": "测试部"
    },
    {
        "id": 6,
        "name": "后勤保障部"
    },
    {
        "id": 7,
        "name": "鸡犬不宁部"
    }
]
```

# 总结

- 其实使用spring boot-1.x版本和2.x版本操作基本是一致的
- 注解方便单表的操作，xml适用于复杂的操作，各有优点，没有最好的，只有最合适的

完整代码链接[github](https://github.com/flower-face/spring-boot-samples)

# 参考

[mybatis](http://www.mybatis.org/mybatis-3/zh/index.html)