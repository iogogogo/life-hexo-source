---
title: 当Gson遇上Java8中的日期API
date: 2020-06-23 21:45:00
tags:
	- Gson
	- Java8
categories: Gson
---



&emsp;Java8开始，JDK中提供了一组新的日期`API`，当我们需要序列化数据成json时，经常会用到`Gson`。当`Java8`中的日期`API`遇上`Gson`时，能否按照预期的想法正常的处理我们的数据呢？



## 使用Gson序列化与反序列化

```java
@Test
public void test() {
  LocalDateTime dateTime = LocalDateTime.now();
  LocalDate date = LocalDate.now();

  Gson gson = new Gson();

  String json = gson.toJson(dateTime);

  log.info("dateTime Serialization:{}", json);
  log.info("dateTime Deserialization:{}", gson.fromJson(json, LocalDateTime.class));

  System.out.println();

  json = gson.toJson(date);
  log.info("date Serialization:{}", json);
  log.info("date Deserialization:{}", gson.fromJson(json, LocalDate.class));
}
```



### LocalDateTime

- 序列化的结果

```verilog
dateTime Serialization:{"date":{"year":2020,"month":6,"day":23},"time":{"hour":21,"minute":58,"second":20,"nano":987000000}}
```



- 反序列化结果

```verilog
dateTime Deserialization:2020-06-23T21:58:20.987
```



### LocalDate

- 序列化的结果

```verilog
date Serialization:{"year":2020,"month":6,"day":23}
```



- 反序列化结果

```verilog
date Deserialization:2020-06-23
```



我们会发现，序列化的结果不是我们想要的，正常应该是一个`ISO`格式的时间才对，但是却成了一个`JsonObject`。很显然这是不满足我们需求的。



## 分析序列化结果不是`ISO`格式的原因

通过源码分析，`LocalDateTime`中引用了`LocalDate`

```java
/**
* The date part.
*/
private final LocalDate date;
/**
* The time part.
*/
private final LocalTime time;
```



其中`LocalDate`和`LocalTime`的部分源码如下

- LocalDate

```java
/**
* The year.
*/
private final int year;
/**
* The month-of-year.
*/
private final short month;
/**
* The day-of-month.
*/
private final short day;
```

- LocalTime

```java
/**
* The hour.
*/
private final byte hour;
/**
* The minute.
*/
private final byte minute;
/**
* The second.
*/
private final byte second;
/**
* The nanosecond.
*/
private final int nano;
```



&emsp;可以看到，`LocalDate`和`LocalTime`，我们序列化`Java8`中的日期`API`，实际上是把成员变量序列化，是正常的一个序列化对象的逻辑。但是我们肯定是不希望这样的结果，对我们来说并不是特别友好。那么怎么解决这个问题呢？



## 自定义`Gson`的`Adapter`解决该问题

`Gson`本身给我们提供了各种各样的配置，其中有一个就是可以自定义序列化或者反序列化的`Adapter`，那么既然现在我们序列化不是我们想要的结果，就可以通过自定义`Adapter`来解决这个问题，废话不说，直接上代码演示。





### 自定义LocalDateAdapter 

```java
/**
* Created by tao.zeng on 2020/6/23.
* <p>
* 处理LocalDate的序列化与反序列化
*/
public final static class LocalDateAdapter implements JsonSerializer<LocalDate>, JsonDeserializer<LocalDate> {

    @Override
    public JsonElement serialize(LocalDate date, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(date.format(DateTimeFormatter.ISO_LOCAL_DATE));
    }

    @Override
    public LocalDate deserialize(JsonElement element, Type type, JsonDeserializationContext context) throws JsonParseException {
        String timestamp = element.getAsJsonPrimitive().getAsString();
        return LocalDate.parse(timestamp, DateTimeFormatter.ISO_LOCAL_DATE);
    }
}
```





### 自定义LocalDateTimeAdapter

```java
/**
* Created by tao.zeng on 2020/6/23.
* <p>
* 处理LocalDateTime序列化与反序列化
*/
public final static class LocalDateTimeAdapter implements JsonSerializer<LocalDateTime>, JsonDeserializer<LocalDateTime> {

    @Override
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





### 使用自定义Adapter

既然我们现在自定义了`Adapter`，那么在使用时就需要将它注册到`gson`对象中去。在创建`gson`对象时就不能直接使用`Gson gson = new Gson()`，而是要使用`GsonBuilder`去进行构建。

```java
// 实例化gson对象时注册Adapter
Gson gson = new GsonBuilder()
                .registerTypeAdapter(LocalDate.class, new JsonParse.LocalDateAdapter())
                .registerTypeAdapter(LocalDateTime.class, new JsonParse.LocalDateTimeAdapter())
                .create();
```



上面我们的代码序列化与反序列时就使用注册了Adapter的gson对象即可

```java
@Test
public void test() {
  LocalDateTime dateTime = LocalDateTime.now();
  LocalDate date = LocalDate.now();

  Gson gson = new GsonBuilder()
    .registerTypeAdapter(LocalDate.class, JsonParse.LocalDateAdapter.class)
    .registerTypeAdapter(LocalDateTime.class, JsonParse.LocalDateTimeAdapter.class)
    .create();

  String json = gson.toJson(dateTime);

  log.info("dateTime Serialization:{}", json);
  log.info("dateTime Deserialization:{}", gson.fromJson(json, LocalDateTime.class));

  System.out.println();

  json = gson.toJson(date);
  log.info("date Serialization:{}", json);
  log.info("date Deserialization:{}", gson.fromJson(json, LocalDate.class));
}
```



#### LocalDateTime

```verilog
dateTime Serialization:"2020-06-23T22:22:10.816"
dateTime Deserialization:2020-06-23T22:22:10.816
```

#### LocalDate

```verilog
date Serialization:"2020-06-23"
date Deserialization:2020-06-23
```



可以看到，在自定义Adapter以后，序列化的结果就是我们想要的`ISO`类型，当然你也可以根据自己的需求将日期格式序列话成自己想要的任意格式。





## Json序列化与反序列化工具类

分享一个日常使用工具类，除了上文说到的关于日期处理的解决方案，还有另外一个问题的解决方案。这个留着下一篇文章讲。



```java
package com.iogogogo.util;

import com.google.gson.*;
import com.google.gson.internal.LinkedTreeMap;
import com.google.gson.reflect.TypeToken;
import lombok.extern.slf4j.Slf4j;

import java.lang.reflect.Type;
import java.nio.charset.StandardCharsets;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * Created by tao.zeng on 2020/6/23.
 */
@Slf4j
public class JsonParse {

    public static Type MAP_STR_OBJ_TYPE = new TypeToken<Map<String, Object>>() {
    }.getType();

    public static Gson GSON = new GsonBuilder()
            .registerTypeAdapter(MAP_STR_OBJ_TYPE, new MapDeserializerDoubleAsIntFix())
            .registerTypeAdapter(LocalDate.class, new LocalDateAdapter())
            .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeAdapter())
            .create();


    /**
     * To json string.
     *
     * @param bean the bean
     * @return the string
     */
    public static String toJson(Object bean) {
        return GSON.toJson(bean);
    }

    /**
     * To json string.
     *
     * @param builder the builder
     * @param bean    the bean
     * @return the string
     */
    public static String toJson(GsonBuilder builder, Object bean) {
        return builder.create().toJson(bean);
    }

    /**
     * Parse t.
     *
     * @param <T>  the type parameter
     * @param json the json
     * @param clz  the clz
     * @return the t
     */
    public static <T> T parse(String json, Class<T> clz) {
        return GSON.fromJson(json, clz);
    }

    /**
     * Parse t.
     *
     * @param <T>     the type parameter
     * @param builder the builder
     * @param json    the json
     * @param clz     the clz
     * @return the t
     */
    public static <T> T parse(GsonBuilder builder, String json, Class<T> clz) {
        return builder.create().fromJson(json, clz);
    }

    /**
     * Parse t.
     *
     * @param <T>  the type parameter
     * @param json the json
     * @param type the type
     * @return the t
     */
    public static <T> T parse(String json, Type type) {
        return GSON.fromJson(json, type);
    }

    /**
     * Parse t.
     *
     * @param <T>     the type parameter
     * @param builder the builder
     * @param json    the json
     * @param type    the type
     * @return the t
     */
    public static <T> T parse(GsonBuilder builder, String json, Type type) {
        return builder.create().fromJson(json, type);
    }

    /**
     * To json bytes byte [ ].
     *
     * @param value the value
     * @return the byte [ ]
     */
    public static byte[] toJsonBytes(Object value) {
        return toJson(value).getBytes(StandardCharsets.UTF_8);
    }


    /**
     * Created by tao.zeng on 2020/6/4.
     * <p>
     * 处理LocalDate的序列化与反序列化
     */
    public final static class LocalDateAdapter implements JsonSerializer<LocalDate>, JsonDeserializer<LocalDate> {

        @Override
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
     * Created by tao.zeng on 2020/6/4.
     * <p>
     * 处理LocalDateTime序列化与反序列化
     */
    public final static class LocalDateTimeAdapter implements JsonSerializer<LocalDateTime>, JsonDeserializer<LocalDateTime> {

        @Override
        public JsonElement serialize(LocalDateTime date, Type typeOfSrc, JsonSerializationContext context) {
            return new JsonPrimitive(date.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
        }

        @Override
        public LocalDateTime deserialize(JsonElement element, Type type, JsonDeserializationContext context) throws JsonParseException {
            String timestamp = element.getAsJsonPrimitive().getAsString();
            return LocalDateTime.parse(timestamp, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
        }
    }

    /**
     * Created by tao.zeng on 2020/6/4.
     * <p>
     * https://gist.github.com/xingstarx/5ddc14ff6ca68ba4097815c90d1c47cc
     * <p>
     * https://stackoverflow.com/questions/36508323/how-can-i-prevent-gson-from-converting-integers-to-doubles/36529534#36529534
     * <p>
     * <p>
     * 解决json数据转换为map结构的时候，会出现int变成double的问题
     */
    public final static class MapDeserializerDoubleAsIntFix implements JsonDeserializer<Map<String, Object>> {

        @SuppressWarnings("unchecked")
        @Override
        public Map<String, Object> deserialize(JsonElement element, Type type, JsonDeserializationContext context) throws JsonParseException {
            return (Map<String, Object>) read(element);
        }

        private Object read(JsonElement in) {
            if (in.isJsonArray()) {
                List<Object> list = new ArrayList<>();
                JsonArray arr = in.getAsJsonArray();
                for (JsonElement anArr : arr) {
                    list.add(read(anArr));
                }
                return list;
            } else if (in.isJsonObject()) {
                Map<String, Object> map = new LinkedTreeMap<>();
                JsonObject obj = in.getAsJsonObject();
                Set<Map.Entry<String, JsonElement>> entitySet = obj.entrySet();
                for (Map.Entry<String, JsonElement> entry : entitySet) {
                    map.put(entry.getKey(), read(entry.getValue()));
                }
                return map;
            } else if (in.isJsonPrimitive()) {
                JsonPrimitive prim = in.getAsJsonPrimitive();
                if (prim.isBoolean()) {
                    return prim.getAsBoolean();
                } else if (prim.isString()) {
                    return prim.getAsString();
                } else if (prim.isNumber()) {
                    Number num = prim.getAsNumber();
                    // here you can handle double int/long values
                    // and return any type you want
                    // this solution will transform 3.0 float to long values
                    if (Math.ceil(num.doubleValue()) == num.longValue())
                        return num.longValue();
                    else {
                        return num.doubleValue();
                    }
                }
            }
            return null;
        }
    }
}

```

