---
title: Java8 stream-实战
date: 2019-03-06 17:03:21
tags: Java8
categories: Java8
---

Java8在日常编码中感触最深的无非就是steam和lambda表达式以及新的时间api，在此之前，集合处理一直是不太方便且性能较低，第二个是时间api不好用，一般配合joda-time这个库配合使用，下面介绍一些在Java8中常用的stream处理

## List转换成Map

```java
/**
  * toMap 如果集合中重复的key 可能会抛出异常 Duplicate key...
  * apply1、apply2的ID都为1 既m1,m2的ID都为1
  * 可以用(m1,m2)->m1 来设置 如果有重复的key 保留m1 舍弃m2
  */
Map<String, Model> modelMap = modelList
                .parallelStream()
                .collect(Collectors.toMap(Model::getId, x -> x, (m1, m2) -> m1));
```



## groupBy以后求最大值或者最小值

http://www.java2s.com/Tutorials/Java_Streams/Example/Group/Get_max_value_in_each_group.htm

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Model {
    String hostname;

    String ip;

    int num;
}

@Test
public void test() {
    List<Model> models = new ArrayList<>();

    models.add(new Model("vm_11_11", "192.168.1.0", 1));
    models.add(new Model("vm_11_11", "192.168.1.1", 2));
    models.add(new Model("vm_11_11", "192.168.1.2", 3));
    models.add(new Model("vm_11_11", "192.168.1.3", 4));
    models.add(new Model("vm_11_11", "192.168.1.2", 5));
    models.add(new Model("vm_11_12", "192.168.1.3", 6));
    models.add(new Model("vm_11_12", "192.168.1.6", 7));
    models.add(new Model("vm_11_12", "192.168.1.7", 8));
    models.add(new Model("vm_11_12", "192.168.1.8", 9));
    models.add(new Model("vm_11_12", "192.168.1.9", 10));
    models.add(new Model("vm_11_13", "192.168.1.10", 11));
    models.add(new Model("vm_11_13", "192.168.1.11", 12));
    models.add(new Model("vm_11_13", "192.168.1.11", 13));
    models.add(new Model("vm_11_13", "192.168.1.12", 14));
    models.add(new Model("vm_11_13", "192.168.1.11", 15));
    models.add(new Model("vm_11_13", "192.168.1.15", 16));
    models.add(new Model("vm_11_16", "192.168.1.16", 17));
    models.add(new Model("vm_11_16", "192.168.1.17", 18));
    models.add(new Model("vm_11_16", "192.168.1.18", 19));
    models.add(new Model("vm_11_16", "192.168.1.19", 20));
    models.add(new Model("vm_11_16", "192.168.1.20", 21));
    models.add(new Model("vm_11_16", "192.168.1.21", 22));
    models.add(new Model("vm_11_16", "192.168.1.22", 23));


    Map<String, Model> map = models
    .parallelStream()
    .collect(Collectors.toMap(model -> model.getHostname() + "_" + model.getIp(),
    Function.identity(),
    (Model d1, Model d2) -> d1.getOrder() < d2.getOrder() ? d1 : d2));

    log.info("map:{}", map);
}
```



## Scala中使用group求出最大值或者最小值

```scala
object CollectionApp {

  def main(args: Array[String]): Unit = {
    val log = LoggerFactory.getLogger(getClass)

    import scala.collection.JavaConverters._

    val models: util.List[Model] = new util.ArrayList[Model]()

    models.add(Model("vm_11_11", "192.168.1.0", 1))
    models.add(Model("vm_11_11", "192.168.1.1", 2))
    models.add(Model("vm_11_11", "192.168.1.2", 3))
    models.add(Model("vm_11_11", "192.168.1.3", 4))
    models.add(Model("vm_11_11", "192.168.1.2", 5))
    models.add(Model("vm_11_12", "192.168.1.3", 6))
    models.add(Model("vm_11_12", "192.168.1.6", 7))
    models.add(Model("vm_11_12", "192.168.1.7", 8))
    models.add(Model("vm_11_12", "192.168.1.8", 9))
    models.add(Model("vm_11_12", "192.168.1.9", 10))
    models.add(Model("vm_11_13", "192.168.1.10", 11))
    models.add(Model("vm_11_13", "192.168.1.11", 12))
    models.add(Model("vm_11_13", "192.168.1.11", 13))
    models.add(Model("vm_11_13", "192.168.1.12", 14))
    models.add(Model("vm_11_13", "192.168.1.11", 15))
    models.add(Model("vm_11_13", "192.168.1.15", 16))
    models.add(Model("vm_11_16", "192.168.1.16", 17))
    models.add(Model("vm_11_16", "192.168.1.17", 18))
    models.add(Model("vm_11_16", "192.168.1.18", 19))
    models.add(Model("vm_11_16", "192.168.1.19", 20))
    models.add(Model("vm_11_16", "192.168.1.20", 21))
    models.add(Model("vm_11_16", "192.168.1.21", 22))
    models.add(Model("vm_11_16", "192.168.1.22", 23))

    val list = models.asScala

    val res = list.groupBy(x => x.hostname + x.ip).map {
      case (k, v) =>
        (k, v.map(_.order).min)
    }

    log.info(s"$res")
  }
}

case class Model(hostname: String, ip: String, order: Int)
```

## 批处理

```java
package com.iogogog.util;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections4.CollectionUtils;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;

/**
 * Created by tao.zeng on 2019-03-09.
 */
@Slf4j
public class BatchProcess {

    public static void process(Collection<?> totalList, int batchSize, BatchProcessListener batchProcessListener) throws Exception {
        if (CollectionUtils.isEmpty(totalList)) return;

        if (batchProcessListener == null) {
            throw new RuntimeException("没有批处理监听器!");
        }

        Iterator<?> iterator = totalList.parallelStream().iterator();
        int i = 0;
        List<Object> list = new ArrayList<>(1024);
        while (iterator.hasNext()) {
            Object next = iterator.next();
            list.add(next);
            if ((i + 1) % batchSize == 0 || i == (totalList.size() - 1)) {
                // process
                batchProcessListener.onProcess(list);
                log.debug("batchSize:{} processSize:{} ", batchSize, list.size());
                list.clear();
            }
            i++;
        }
    }

    public interface BatchProcessListener {
        void onProcess(List<?> list) throws Exception;
    }
}

```

