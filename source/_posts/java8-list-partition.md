---
title: Java8 list分页
date: 2021-04-11 15:57:20
tags: Java
categories:
---

日常开发中经常遇到list分页的需求，在Java8之前都是使用subList进行截取，在Java8以后，有了新的选择，那就是使用stream，原理很简单，就是使用stream提供的`skip`和`limit`方法。

这里的`ListUtils`是一个接口，在Java8之前接口不能有方法实现，在Java8以后，接口中可以存在`static`方法并且完成方法实现，另外也可以使用`default`关键字来完成一个方法实现，可以省略我们写`public static`关键字（目前没发现其他好处😂）

## 封装工具类

```java
package com.iogogogo.util;

import lombok.Data;

import java.io.Serializable;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

/**
 * Created by tao.zeng on 2021/4/8.
 */
public interface ListUtils {

    static <T> PageWrapper<T> partition(int pageNo, int pageSize, List<T> records) {
        if (pageNo <= 0) pageNo = 1;

        // 计算偏移量
        int offset = (pageNo - 1) * pageSize;

        // 总算总记录数
        int totalSize = records.size();

        if (offset > totalSize) return new PageWrapper<>(pageNo, pageSize, totalSize, Collections.emptyList());

        // 分页结果数据
        List<T> collect = records.stream().skip(offset).limit(pageSize).collect(Collectors.toList());

        return new PageWrapper<>(pageNo, pageSize, totalSize, collect);
    }


    /**
     * The type Page wrapper.
     *
     * @param <T> the type parameter
     */
    /*
     *public static void main(String[] args) {
        int totalSize = 11;//数据总量
        int pageSize = 3;//一页显示条数
        int totalPage;//总页数
        
        // 几种页数计算方法
        totalPage = totalSize / pageSize;
        if (totalSize % pageSize != 0) {
            totalPage++;
        }
        System.out.println(totalPage);//此方法容易理解
        System.out.println((totalSize - 1) / pageSize + 1);//此方法使用较多
        System.out.println((totalSize + pageSize - 1) / pageSize);
    }
     */
    @Data
    class PageWrapper<T> implements Serializable {
        /**
         * 当前页，每页显示size
         */
        private int pageNo, pageSize;

        /**
         * 总记录数，总页数
         */
        private long totalSize, totalPage;

        /**
         * 分页结果数据
         */
        private List<T> records;

        /**
         * Instantiates a new Page wrapper.
         *
         * @param pageNo    the page no
         * @param pageSize  the page size
         * @param totalSize the total size
         * @param records   the records
         */
        public PageWrapper(int pageNo, int pageSize, long totalSize, List<T> records) {
            this.setPageNo(pageNo);
            this.pageSize = pageSize;
            this.records = records;
            this.totalSize = totalSize;
            this.totalPage = (totalSize - 1) / pageSize + 1;
        }

        /**
         * Sets page no.
         *
         * @param pageNo the page no
         */
        public void setPageNo(int pageNo) {
            if (pageNo <= 0) this.pageNo = 1;
            this.pageNo = pageNo;
        }
    }
}

```



## 单元测试

```java
package com.iogogogo.util;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

/**
 * Created by tao.zeng on 2021/4/11.
 */
class ListUtilsTest {

    @Test
    public void test() {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            list.add(UUID.randomUUID().toString());
        }
        ListUtils.PageWrapper<String> wrapper = ListUtils.partition(0, 5, list);

        Assertions.assertEquals(wrapper.getPageNo(), 1);
        Assertions.assertEquals(wrapper.getPageSize(), 5);
        Assertions.assertEquals(wrapper.getTotalPage(), 4);
        Assertions.assertEquals(wrapper.getRecords().size(), 5);

        wrapper = ListUtils.partition(30, 20, list);

        Assertions.assertEquals(wrapper.getPageNo(), 30);
        Assertions.assertEquals(wrapper.getPageSize(), 20);
        Assertions.assertEquals(wrapper.getTotalPage(), 1);
        Assertions.assertEquals(wrapper.getRecords().size(), 0);

    }
}
```

