---
title: Apache CollectionUtils 常用方法总结
date: 2019-03-01 10:09:21
tags: Java
cover: false
categories: Java
---

CollectionUtils在真实项目中，是一个非常好用的工具类，使用非常频繁。它可以使代码更加简洁和安全。

# 依赖

- maven

```xml
<!-- https://mvnrepository.com/artifact/commons-collections/commons-collections -->
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.1</version>
</dependency>

```

- gradle

```groovy
// https://mvnrepository.com/artifact/commons-collections/commons-collections
compile group: 'commons-collections', name: 'commons-collections', version: '3.2.1'
```

# 常用方法

### 非空判断

```java
// 判断集合是否为空:
CollectionUtils.isEmpty(null): true
CollectionUtils.isEmpty(new ArrayList()): true　　
CollectionUtils.isEmpty({a,b}): false

// 判断集合是否不为空:
CollectionUtils.isNotEmpty(null): false
CollectionUtils.isNotEmpty(new ArrayList()): false
CollectionUtils.isNotEmpty({a,b}): true
```

### 并集、交集、交集的补集、差集

```java
@Test
public void test() {
	List<String> list1 = Stream.of("a", "b", "c", "d", "e").parallel().collect(Collectors.toList());
	List<String> list2 = Stream.of("a", "f", "C", "e", "g","z").parallel().collect(Collectors.toList());

    // 并集
    Collection union = CollectionUtils.union(list1, list2);
    log.info("union:{}", union);

    // 交集
    Collection intersection = CollectionUtils.intersection(list1, list2);
    log.info("intersection:{}", intersection);

    // 交集的补集（析取）
    Collection disjunction = CollectionUtils.disjunction(list1, list2);
    log.info("disjunction:{}", disjunction);

    // 差集（扣除）list1扣除list2
    Collection subtract = CollectionUtils.subtract(list1, list2);
    log.info("subtract:{}", subtract);
}

// 结果
union:[a, b, c, C, d, e, f, g, z]
intersection:[a, e]
disjunction:[b, c, C, d, f, g, z]
subtract:[b, c, d]
```

### 判断相等

```java
@Test
public void testEqual() {
    List<String> list1 = Stream.of("a", "b", "c", "d", "e").parallel().collect(Collectors.toList());
    List<String> list2 = Stream.of("a", "f", "C", "e", "g", "z").parallel().collect(Collectors.toList());

    // 比较值 false
    log.info("isEqualCollection:{}", CollectionUtils.isEqualCollection(list1, list2));

    
    List<Integer> list3 = Stream.of(1, 2, 3, 4).parallel().collect(Collectors.toList());
    List<Integer> list4 = Stream.of(1, 2, 3, 4).parallel().collect(Collectors.toList());
    // 比较值 true
    log.info("isEqualCollection:{}", CollectionUtils.isEqualCollection(list3, list4));
    
    class Person {
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @EqualsAndHashCode(callSuper = true)
    class Boy extends Person {
    String name;
    }

    List<Person> boy1 = new ArrayList<>();
    boy1.add(new Boy("阿牛"));
    List<Person> boy2 = new ArrayList<>();
    boy2.add(new Boy("阿牛"));

    // 比较集合中不同对象 false
    log.info("isEqualCollection:{}", CollectionUtils.isEqualCollection(boy1, boy2));


    Boy boy = new Boy();
    List<Boy> boy3 = new ArrayList<>();
    boy3.add(boy);
    List<Boy> boy4 = new ArrayList<>();
    boy4.add(boy);
    // 比较集合中相同对象 true
    log.info("isEqualCollection:{}", CollectionUtils.isEqualCollection(boy3, boy4));
}
```



### 不可变集合

```java
@Test
public void testUnmodifiable() {
    Collection<String> list = new ArrayList<>();
    // 抓换为不可变集合，添加数据会报错
    Collection<String> collection = CollectionUtils.unmodifiableCollection(list);
    collection.add("a");
    collection.add("b");
    collection.add("c");

    log.info("collection:{}", collection);
}


// Collections.unmodifiableCollection可以得到一个集合的镜像，它的返回结果是不可直接被改变，否则会提示错误
java.lang.UnsupportedOperationException
at org.apache.commons.collections.collection.UnmodifiableCollection.add(UnmodifiableCollection.java:75)
```


