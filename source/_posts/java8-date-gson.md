---
title: Java8 中的日期类序列化问题
date: 2020-01-07 21:59:27
tags:
 - Java8
 - Gson
cover: false
categories: Java8
---

自从习惯了使用Java8 中的日期类以后，就已经完全抛弃了`java.sql`、`java.util`中的日期处理类，但是Java8中的日期类在序列化与反序列化时不是我们正常看到的标准格式，这里记录一下如何使用`Gson`进行序列化和反序列化时正常对Java8的日期类进行处理



## 默认序列化结果

这里写了一个很简单的test，对`LocalDate`、`LocalDateTime`进行序列化与反序列化

```java
package com.iogogogo;

import com.google.gson.Gson;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.Serializable;
import java.time.LocalDate;
import java.time.LocalDateTime;

/**
 * Created by tao.zeng on 2020-01-07.
 */
@Slf4j
public class Java8DateTests {

    @Test
    public void test1() {
        DateMetric dateMetric = new DateMetric();
        dateMetric.setLocalDate(LocalDate.now());
        dateMetric.setLocalDateTime(LocalDateTime.now());

        // serializer
        Gson gson = new Gson();
        String json = gson.toJson(dateMetric);
        log.info("serializer:{}", json);

        // deserializer
        log.info("deserializer:{}", gson.fromJson(json, DateMetric.class));
    }

    @Data
    private class DateMetric implements Serializable {

        private LocalDate localDate;

        private LocalDateTime localDateTime;
    }
}

```

查看结果，可以看到反序列化的结果是我们自己想要的，但是序列化的结果并不是我们想要的`yyyy-MM-dd HH:mm:ss` 这种格式，那么这时候就需要用到`GsonBuilder`来注册`registerTypeAdapter`

```verilog
22:09:01.345 [main] INFO com.iogogogo.Java8DateTests - serializer:{"localDate":{"year":2020,"month":1,"day":7},"localDateTime":{"date":{"year":2020,"month":1,"day":7},"time":{"hour":22,"minute":9,"second":1,"nano":271000000}}}

22:09:01.356 [main] INFO com.iogogogo.Java8DateTests - deserializer:Java8DateTests.DateMetric(localDate=2020-01-07, localDateTime=2020-01-07T22:09:01.271)

```



## 自定义序列化与反序列化的Adapter

```java
/**
     * 处理LocalDate的序列化与反序列化
     */
final static class LocalDateAdapter implements JsonSerializer<LocalDate>, JsonDeserializer<LocalDate> {

  public JsonElement serialize(LocalDate date, Type typeOfSrc, JsonSerializationContext context) {
    return new JsonPrimitive(date.format(DateTimeFormatter.ISO_LOCAL_DATE));
  }

  @Override
  public LocalDate deserialize(JsonElement element, Type type, JsonDeserializationContext context) throws JsonParseException {
    String timestamp = element.getAsJsonPrimitive().getAsString();
    return LocalDate.parse(timestamp, DateTimeFormatter.ISO_LOCAL_DATE);
  }
}

/**
     * 处理LocalDateTime序列化与反序列化
     */
final static class LocalDateTimeAdapter implements JsonSerializer<LocalDateTime>, JsonDeserializer<LocalDateTime> {

  public JsonElement serialize(LocalDateTime date, Type typeOfSrc, JsonSerializationContext context) {
    return new JsonPrimitive(date.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
  }

  @Override
  public LocalDateTime deserialize(JsonElement element, Type type, JsonDeserializationContext context) throws JsonParseException {
    String timestamp = element.getAsJsonPrimitive().getAsString();
    return LocalDateTime.parse(timestamp, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
  }
}
```

### 使用GsonBuilder创建gson对象

```java
@Test
public void test2() {
  DateMetric dateMetric = new DateMetric();
  dateMetric.setLocalDate(LocalDate.now());
  dateMetric.setLocalDateTime(LocalDateTime.now());

  // serializer
  Gson gson = new GsonBuilder()
    .registerTypeAdapter(LocalDate.class, new LocalDateAdapter())
    .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeAdapter())
    .create();

  String json = gson.toJson(dateMetric);
  log.info("serializer:{}", json);

  // deserializer
  log.info("deserializer:{}", gson.fromJson(json, DateMetric.class));
}
```

结果

```verilog
22:16:15.738 [main] INFO com.iogogogo.Java8DateTests - serializer:{"localDate":"2020-01-07","localDateTime":"2020-01-07T22:16:15.662"}

22:16:15.745 [main] INFO com.iogogogo.Java8DateTests - deserializer:Java8DateTests.DateMetric(localDate=2020-01-07, localDateTime=2020-01-07T22:16:15.662)

```



## 输出json进行格式化处理

平时输出的json都是一行文本，如果需要对json的输出结果需要进行格式化处理的话，也可以使用`GsonBuilder`的`setPrettyPrinting()`方法

```java
@Test
public void test2() {
  DateMetric dateMetric = new DateMetric();
  dateMetric.setLocalDate(LocalDate.now());
  dateMetric.setLocalDateTime(LocalDateTime.now());

  // serializer
  Gson gson = new GsonBuilder()
    .setPrettyPrinting() // 设置输出格式格式化显示
    .registerTypeAdapter(LocalDate.class, new LocalDateAdapter())
    .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeAdapter())
    .create();

  String json = gson.toJson(dateMetric);
  log.info("serializer:\n{}", json);

  // deserializer
  log.info("deserializer:{}", gson.fromJson(json, DateMetric.class));
}
```

输出结果

```verilog
22:21:03.307 [main] INFO com.iogogogo.Java8DateTests - serializer:
{
  "localDate": "2020-01-07",
  "localDateTime": "2020-01-07T22:21:03.218"
}

22:21:03.313 [main] INFO com.iogogogo.Java8DateTests - deserializer:Java8DateTests.DateMetric(localDate=2020-01-07, localDateTime=2020-01-07T22:21:03.218)

```





## 对已有的json进行格式化处理

如果我们的json文本已经存在，那我我们想通过gson进行格式化展示该怎么做呢？

这里做法也很简单，不过`JsonObject`和`JsonArray`略有区别，不过都是需要将json读取成`JsonObject`或者`JsonArray`对象

### JsonObject 格式化

原始JsonObject数据

```json
{"localDate":{"year":2020,"month":1,"day":7},"localDateTime":{"date":{"year":2020,"month":1,"day":7},"time":{"hour":22,"minute":9,"second":1,"nano":271000000}}}
```

```java
@Test
public void jsonObjectFormatter() {
  JsonObject jsonObject = JsonParser.parseString(JSON_STRING).getAsJsonObject();
  Gson gson = new GsonBuilder().setPrettyPrinting().create();
  System.out.println(gson.toJson(jsonObject));
}
```

输出结果：

```json
{
  "localDate": {
    "year": 2020,
    "month": 1,
    "day": 7
  },
  "localDateTime": {
    "date": {
      "year": 2020,
      "month": 1,
      "day": 7
    },
    "time": {
      "hour": 22,
      "minute": 9,
      "second": 1,
      "nano": 271000000
    }
  }
}
```





### JsonArray 格式化

原始JsonArray数据

```json
[{"localDate":{"year":2020,"month":1,"day":7},"localDateTime":{"date":{"year":2020,"month":1,"day":7},"time":{"hour":22,"minute":9,"second":1,"nano":271000000}}}]
```



```java
@Test
public void jsonArrayFormatter() {
  JsonArray jsonArray = JsonParser.parseString(JSON_STRING_ARRAY).getAsJsonArray();
  Gson gson = new GsonBuilder().setPrettyPrinting().create();
  System.out.println(gson.toJson(jsonArray));
}

```

输出结果：

```json
[
  {
    "localDate": {
      "year": 2020,
      "month": 1,
      "day": 7
    },
    "localDateTime": {
      "date": {
        "year": 2020,
        "month": 1,
        "day": 7
      },
      "time": {
        "hour": 22,
        "minute": 9,
        "second": 1,
        "nano": 271000000
      }
    }
  }
]

```



## 总结

本文主要介绍了使用Gson对Java8中的日志进行序列化与反序列化格式的处理，以及使用gson对json进行格式化的输出。

参考地址：[Serialize Java 8 LocalDate as yyyy-mm-dd with Gson](https://stackoverflow.com/questions/39192945/serialize-java-8-localdate-as-yyyy-mm-dd-with-gson)

完整代码：[Java8DateTests](https://github.com/iogogogo/life-example/blob/master/example-kafka/src/test/java/com/iogogogo/Java8DateTests.java)





