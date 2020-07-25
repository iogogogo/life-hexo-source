---
title: Map对key或者value进行排序
date: 2020-07-25 12:21:58
tags: map
categories: Java
---

开发中偶尔会遇到一些比较特殊的需求，比如对一个map进行排序，并且是对key或者value进行排序，那么我们可以用Java8中提供的`stream`来进行实现





## 排序工具类

封装排序工具类


```java
package com.iogogogo.common.util;

import com.google.common.collect.Lists;
import com.google.common.collect.Maps;

import java.lang.reflect.Field;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Map;

/**
 * Created by tao.zeng on 2020/5/25.
 */
public class MapSortUtils {

    /**
     * Sort by key map.
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the map
     */
    public static <K extends Comparable<? super K>, V> Map<K, V> sortByKey(Map<K, V> map) {
        // 这里使用了guava简化了map对象的创建，没有guava直接使用 new LinkedHashMap<>();
        Map<K, V> result = Maps.newLinkedHashMap();
        map.entrySet().stream()
                .sorted(Map.Entry.comparingByKey()).forEachOrdered(e -> result.put(e.getKey(), e.getValue()));
        return result;
    }

    /**
     * Sort reversed by key map.
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the map
     */
    public static <K extends Comparable<? super K>, V> Map<K, V> sortReversedByKey(Map<K, V> map) {
        Map<K, V> result = Maps.newLinkedHashMap();
        map.entrySet().stream()
                .sorted(Map.Entry.<K, V>comparingByKey()
                        .reversed()).forEachOrdered(e -> result.put(e.getKey(), e.getValue()));
        return result;
    }

    /**
     * Sort by value map.
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the map
     */
    public static <K, V extends Comparable<? super V>> Map<K, V> sortByValue(Map<K, V> map) {
        Map<K, V> result = Maps.newLinkedHashMap();
        map.entrySet().stream()
                .sorted(Map.Entry.comparingByValue()).forEachOrdered(e -> result.put(e.getKey(), e.getValue()));
        return result;
    }

    /**
     * Sort reversed by value map.
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the map
     */
    public static <K, V extends Comparable<? super V>> Map<K, V> sortReversedByValue(Map<K, V> map) {
        Map<K, V> result = Maps.newLinkedHashMap();
        map.entrySet().stream()
                .sorted(Map.Entry.<K, V>comparingByValue()
                        .reversed()).forEachOrdered(e -> result.put(e.getKey(), e.getValue()));
        return result;
    }

    /**
     * Gets first key.
     *
     * @param <K> the type parameter
     * @param map the map
     * @return the first key
     */
    public static <K> K getFirstKey(Map<K, ?> map) {
        return map.keySet().iterator().next();
    }

    /**
     * Gets last key.
     *
     * @param <K> the type parameter
     * @param map the map
     * @return the last key
     */
    public static <K> K getLastKey(Map<K, ?> map) {
        Iterator<K> iterator = map.keySet().iterator();
        LinkedList<K> list = Lists.newLinkedList();
        while (iterator.hasNext()) {
            list.add(iterator.next());
        }
        return list.getLast();
    }

    /**
     * Gets first.
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the first
     */
    public static <K, V> Map.Entry<K, V> getFirst(Map<K, V> map) {
        return map.entrySet().iterator().next();
    }

    /**
     * Gets last.
     * <p>
     * 通过反射获取LinkedHashMap中的末尾元素：
     * <p>
     * 时间复杂度O(1)，访问tail属性
     *
     * @param <K> the type parameter
     * @param <V> the type parameter
     * @param map the map
     * @return the last
     * @throws IllegalAccessException the illegal access exception
     * @throws NoSuchFieldException   the no such field exception
     */
    @SuppressWarnings("unchecked")
    public static <K, V> Map.Entry<K, V> getLast(Map<K, V> map) throws IllegalAccessException, NoSuchFieldException {
        Field tail = map.getClass().getDeclaredField("tail");
        tail.setAccessible(true);
        return (Map.Entry<K, V>) tail.get(map);
    }
}
```





## 测试结果

```java
package com.iogogogo.common.tests;

import com.iogogogo.common.util.MapSortUtils;
import com.google.common.collect.Maps;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import java.util.Map;

/**
 * Created by tao.zeng on 2020/7/25.
 */
public class MapSortTest {

    private final static Map<String, Integer> DEF_DATA = Maps.newHashMap();


    @Before
    public void before() {
        DEF_DATA.put("c", 20);
        DEF_DATA.put("a", 54);
        DEF_DATA.put("z", 23);
        DEF_DATA.put("d", 55);
    }


    @Test
    public void testKeySort() {

        // 按照key进行正序排序
        Map<String, Integer> sortByKey = MapSortUtils.sortByKey(DEF_DATA);

        // 获取map中的第一个key
        String firstKey = MapSortUtils.getFirstKey(sortByKey);
        // 获取map中的最后一个key
        String lastKey = MapSortUtils.getLastKey(sortByKey);

        Assert.assertEquals(firstKey, "a");
        Assert.assertEquals(lastKey, "z");


        // 按照key对map进行倒序排序
        Map<String, Integer> reversedByKey = MapSortUtils.sortReversedByKey(DEF_DATA);
        firstKey = MapSortUtils.getFirstKey(reversedByKey);
        lastKey = MapSortUtils.getLastKey(reversedByKey);

        Assert.assertEquals(firstKey, "z");
        Assert.assertEquals(lastKey, "a");
    }


    @Test
    public void testValue() throws NoSuchFieldException, IllegalAccessException {

        // 按照value进行正序排序
        Map<String, Integer> sortByValue = MapSortUtils.sortByValue(DEF_DATA);
        Map.Entry<String, Integer> first = MapSortUtils.getFirst(sortByValue);
        Map.Entry<String, Integer> last = MapSortUtils.getLast(sortByValue);

        Assert.assertEquals(first.getKey(), "c");
        Assert.assertEquals(20, (int) first.getValue());

        Assert.assertEquals(last.getKey(), "d");
        Assert.assertEquals(55, (int) last.getValue());


        // 按照value进行倒序排序
        Map<String, Integer> reversedByValue = MapSortUtils.sortReversedByValue(DEF_DATA);

        // 获取排序以后的第一个个最后一个元素
        first = MapSortUtils.getFirst(reversedByValue);
        last = MapSortUtils.getLast(reversedByValue);

        Assert.assertEquals(first.getKey(), "d");
        Assert.assertEquals(55, (int) first.getValue());

        Assert.assertEquals(last.getKey(), "c");
        Assert.assertEquals(20, (int) last.getValue());
    }
}

```

