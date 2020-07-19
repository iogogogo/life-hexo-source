---
title: MyBatis对象中属性 包含List<String>一对多映射处理方式
date: 2020-07-19 22:37:38
tags: MyBatis
categories: MyBatis
---

在使用MyBatis查询数据库时，经常会有一对多的情况，那么在一对多的情况时，如果是一个`Collection<String>`或者`Collection<Integer>`  类型，那么我们的`ResultMap`该如何定义？



方法很简单，这时候我们就需要使用到构造函数注入了，通过Integer和String的构造函数注入，具体的字段名称自己对好入座即可。



```xml
<resultMap type="User" id="user_map">
    <id property="id" column=""/>
    <result property="username" column="username"/>
    <collection property="age" ofType="int">
        <constructor>
            <arg column="age"/> <!-- 对号入座数据库column名称即可 -->
        </constructor>
    </collection>
    <collection property="authorities" ofType="java.lang.String">
        <constructor>
            <arg column="permission"/>  <!-- 对号入座数据库column名称即可 -->
        </constructor>
     </collection>
</resultMap>
```

