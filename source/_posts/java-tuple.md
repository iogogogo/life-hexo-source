---
title: Java中使用元组
date: 2020-01-09 12:26:39
tags: Java
categories: Java
---

元组（Tuple）是固定数量的不同类型的元素的组合。元组与集合的不同之处在于，元组中的元素类型可以是不同的，而且数量固定。元组的好处在于可以把多个元素作为一个单元传递。如果一个方法需要返回多个值，可以把这多个值作为元组返回，而不需要创建额外的类来表示。根据元素数量的不同，Vavr 总共提供了 Tuple0、Tuple1 到 Tuple8 等 9 个类。每个元组类都需要声明其元素类型。如 Tuple2<String, Integer>表示的是两个元素的元组，第一个元素的类型为 String，第二个元素的类型为 Integer。对于元组对象，可以使用 _1、_2 到 _8 来访问其中的元素。所有元组对象都是不可变的，在创建之后不能更改。

元组通过接口 Tuple 的静态方法 of 来创建。元组类也提供了一些方法对它们进行操作。由于元组是不可变的，所有相关的操作都返回一个新的元组对象。在 清单 1 中，使用 Tuple.of 创建了一个 Tuple2 对象。Tuple2 的 map 方法用来转换元组中的每个元素，返回新的元组对象。而 apply 方法则把元组转换成单个值。其他元组类也有类似的方法。除了 map 方法之外，还有 map1、map2、map3 等方法来转换第 N 个元素；update1、update2 和 update3 等方法用来更新单个元素。

<br/>



Python和Scala语言中有自带元组，Jdk中是没有这个数据类型的，虽然使用数组或者map也能达到想要的效果，但总归是没有元组方便。



比如说一个方法有多个返回值时，虽然可以用`Object[]`或者`Map`进行封装，但是在拆箱的时候类型会缺失，所以还是不太方便。Java中有很多元组库，这里推荐一个我经常使用的[vavr](https://www.vavr.io/)，当然`Tuple`只是这个库中的一小部分，该库提供了强大的函数式编程的能力，可以像写Scala一样写Java



## 引入pom依赖

目前最新的稳定版是`0.10.2`

```xml
<!-- https://mvnrepository.com/artifact/io.vavr/vavr -->
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.2</version>
</dependency>

```



## 常用方法使用

### 创建元组对象

```java
@Test
public void test1() {
  Tuple3<String, LocalDateTime, Long> tuple3 = Tuple.of("TupleTests", LocalDateTime.now(), Long.MAX_VALUE);

  System.out.println(tuple3);
}
```

可以看到我们通过`Tuple.of`创建了一个有三个元素的元组对象，输出结果

```verilog
(TupleTests, 2020-01-09T20:40:26.137, 9223372036854775807)
```



### 更新元组内容

```java
@Test
public void test1() {
  Tuple3<String, LocalDateTime, Long> tuple3 = Tuple.of("TupleTests", LocalDateTime.now(), Long.MAX_VALUE);

  System.out.println(tuple3);

  tuple3 = tuple3.update1("哈哈哈哈");
  System.out.println(tuple3);
}
```

通过`update1-n`方法，可以实现元组内容的更新，更新结果

```verilog
(TupleTests, 2020-01-09T20:42:27.623, 9223372036854775807)
(哈哈哈哈, 2020-01-09T20:42:27.623, 9223372036854775807)
```



### 方法多个返回值使用元组接收

这个需求其实之前在[Java8 处理常见的日期周期](https://iogogogo.github.io/2020/01/04/java8-date-cycle/)这篇文章已经用过了，比如我们需要获取一个周期数据，有开始和结束，那么元组无疑是最好的选择。比如我们需要封装一个方法获取今天的最小时间和最大时间，那么这里就需要两个返回值，虽然我们可以用`Object[]`或者`Map`之类的容器达到我们的效果，但是如果我返回值得类型都不尽想同，那么在获取的时候就要进行类型转换，加到了转换出错的概率，但是使用元组就不会有这个问题。



- 获取一天的开始和结束时间

  ```java
  public Tuple2<LocalDateTime, LocalDateTime> getNowRange() {
    LocalDate now = LocalDate.now();
    // 一天中从零时开始
    LocalDateTime startTime = now.atTime(LocalTime.MIN);
    // 一天到23:59:59秒结束
    LocalDateTime endTime = now.atTime(LocalTime.MAX);
    return Tuple.of(startTime, endTime);
  }
  ```

- 获取返回值

  ```java
  @Test
  public void test2() {
    Tuple2<LocalDateTime, LocalDateTime> tuple2 = getNowRange();
  
    // 这里通过 _n 或者 _n() 获取每一个元组中的一列数据
    System.out.println("开始时间:" + tuple2._1());
    System.out.println("结束时间:" + tuple2._2);
  }
  ```

  我们可以看到`getNowRange`同时有两个放回值，通过`Tuple2`进行保存，过 _n 或者 _n() 获取每一个元组中的一列数据。最后获取的结果为

  ```
  开始时间:2020-01-09T00:00
  结束时间:2020-01-09T23:59:59.999999999
  ```

  



## 总结

以上就是元组最常用的一些方法，其实我理解的元组就是一种列式存储的数据容器，和我们平时用的`Collection`接口下的一些`List`，`Set`不同，他们都是以行的形式存储，有了`Tuple`以后，在日常开发中也多了一种数据在内存中存储的选择。

当然元组只是`vavr`库中的一小部分，更多使用方法还是要看官方文档

参考文章：[使用 Vavr 进行函数式编程](https://www.ibm.com/developerworks/cn/java/j-understanding-functional-programming-4/index.html)