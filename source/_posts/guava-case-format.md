---
title: Guava中的CaseFormat
date: 2020-07-25 13:47:19
tags: Guava
categories: Guava
---

在数据库中，按照阿里巴巴java开发手册，数据库字段全部小写并且按照`_`进行单词之间的分割，但是我们`Java`中的命名风格又是按照驼峰方式，一般返回给前端的`DTO`对象也按照驼峰方式，前端同学在对所有列进行排序时，会按照我们返回给他的`props`发送给后端，这时候后端收到的就是驼峰方式的`props`，但是拿到数据库查询这肯定是一个不存在的字段，那么有什么好的方式去解决这个问题呢？



这时候发现了`guava`中的`CaseFormat`工具类。



 com.google.common.base.CaseFormat是一种实用工具类，以提供不同的ASCII字符格式之间的转换。



## 类声明

```java
@GwtCompatible
public enum CaseFormat
   extends Enum<CaseFormat>
```



## 枚举常量

| 枚举常量 | 说明 |
| ------------ | -------- |
| LOWER_CAMEL | Java变量的命名规则，如 `lowerCamel` |
| LOWER_HYPHEN | 连字符连接变量的命名规则，如 `lower-hyphen` |
| LOWER_UNDERSCORE | C ++变量命名规则，如 `lower_underscore` |
| UPPER_CAMEL | Java和C++类的命名规则，如 `UpperCamel` |
| UPPER_UNDERSCORE | Java和C++常量的命名规则，如 `UPPER_UNDERSCORE` |

## 方法

| 方法                                                         | 说明                                             |
| ------------------------------------------------------------ | ------------------------------------------------ |
| Converter<String,String> converterTo(CaseFormat targetFormat) | 返回一个转换，从这个格式转换targetFormat字符串。 |
| String to(CaseFormat format, String str)                     | 从这一格式指定格式的指定字符串 str 转换。        |
| static CaseFormat valueOf(String name)                       | 返回此类型具有指定名称的枚举常量。               |
| static CaseFormat[] values()                                 | 返回一个包含该枚举类型的常量数组中的顺序被声明。 |

## 测试

```java
@Test
public void test() {
    String appCiNum = CaseFormat.LOWER_CAMEL.to(CaseFormat.LOWER_UNDERSCORE, "appCiNum");
    System.out.println(appCiNum);
    Assert.assertEquals(appCiNum, "app_ci_num");

    appCiNum = CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "app_ci_num");
    System.out.println(appCiNum);
    Assert.assertEquals(appCiNum, "appCiNum");
}
```





只要我们自己定的`DTO`是标准的，那么排序是前端同学就按照我们返回给他的字段在传回来，我们只需要用`CaseFormat`进行转换即可，不然还得依次进行match匹配