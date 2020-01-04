---
title: Java8 处理常见的日期周期
date: 2020-01-04 13:56:18
tags: Java8
cover: true
categories: Java8
---

日常开发中，经常会有获取周、月、季度的开始和结束时间的需求，Java8之前的date类和Calendar结合也可以实现，但是还是比较复杂，下面使用Java8的日期api获取周月季的周期。



## 获取当前是周几

```java
LocalDateTime now = LocalDateTime.now();
System.out.println(now);
System.out.println(now.getDayOfWeek().getValue());

// 输出结果
2020-01-04T14:07:15.870
6
```



## 获取当前周的开始和结束时间

```java
LocalDateTime now = LocalDateTime.now();
System.out.println("当前时间:" + now + "\n");

//当前时间减去今天是周几()
LocalDateTime startTime = now.minusDays(now.getDayOfWeek().getValue());
//当前时间加上（8-今天周几）
LocalDateTime endTime = now.plusDays(8 - now.getDayOfWeek().getValue());
System.out.println("本周开始时间:" + startTime);
System.out.println("本周结束时间:" + endTime);

// 输出结果
当前时间:2020-01-04T14:14:56.110

本周开始时间:2019-12-29T14:14:56.110
本周结束时间:2020-01-06T14:14:56.110
```

但是发现一个问题，以上获取到的结果一周开始是从周天开始的，并不是我们日常的习惯从周一开始，那么要从周一开始可以使用`WeekFields`设置一周的开始是周几

```java
LocalDateTime now = LocalDateTime.now();
System.out.println("当前时间:" + now + "\n");

// 使用WeekFields设置一周的开始为周一
TemporalField fieldISO = WeekFields.of(DayOfWeek.MONDAY, 1).dayOfWeek();
LocalDateTime startTime = now.with(fieldISO, 1);
LocalDateTime endTime = now.with(fieldISO, 7);

System.out.println("本周开始时间:" + startTime);
System.out.println("本周结束时间:" + endTime);

// 输出结果
当前时间:2020-01-04T14:17:13.176

本周开始时间:2019-12-30T14:17:13.176
本周结束时间:2020-01-05T14:17:13.176
```



以上两种方式可以获取到本周的开始和结束时间，但是如果我们需要一周从00点开始，到23:59:59秒结束，又该如何设置呢？

其实也很简单，只需要在`LocalDateTime`对象的 `LocalTime`即可

```java
LocalDateTime now = LocalDateTime.now();
System.out.println("当前时间:" + now + "\n");

// 使用WeekFields设置一周的开始为周一
TemporalField fieldISO = WeekFields.of(DayOfWeek.MONDAY, 1).dayOfWeek();
// 设置开始时间为LocalTime.MIN
LocalDateTime startTime = now.with(fieldISO, 1).with(LocalTime.MIN);
// 设置结束时间为LocalTime.MAX
LocalDateTime endTime = now.with(fieldISO, 7).with(LocalTime.MAX);

System.out.println("本周开始时间:" + startTime);
System.out.println("本周结束时间:" + endTime);

// 输出结果
当前时间:2020-01-04T14:22:01.291

本周开始时间:2019-12-30T00:00
本周结束时间:2020-01-05T23:59:59.999999999
```



## 获取本月的开始和结束时间

如果不需要时间归到00点和23:59:59，则不设置`LocalTime`即可

```java
LocalDateTime now = LocalDateTime.now();
System.out.println("当前时间:" + now + "\n");
LocalDateTime startTime = now.with(TemporalAdjusters.firstDayOfMonth()).with(LocalTime.MIN);
LocalDateTime endTime = now.with(TemporalAdjusters.lastDayOfMonth()).with(LocalTime.MAX);

System.out.println("本月开始时间:" + startTime);
System.out.println("本月结束时间:" + endTime);

// 输出结果
当前时间:2020-01-04T14:27:33.706

本月开始时间:2020-01-01T00:00
本月结束时间:2020-01-31T23:59:59.999999999
```



## 获取一个季度的开始和结果

获取季度的开始和结束Java8并没有提供直接的api使用，但是我们可以使用`TemporalQuery`进行自定义查询获取，而且季度和月度以及周有些许不同，每年的季度开始和结束都是固定的

- 一季度的始终是每年的`01.01-03.31`
- 二季度的始终是每年的`04.01-06.30`
- 三季度的始终是每年的`07.01-09.30`
- 四季度的始终是每年的`10.01-12.31`



废话不多说，直接上代码，先定义了一个`QuarterCycle`类

`getQuarterRange`方法中使用了`Tuple`，主要是为了方便存储开始和结束时间，也可以使用`LocalDateTime[]`存储，并没有特殊意义

### QuarterCycle

```java

import io.vavr.Tuple;
import io.vavr.Tuple2;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Month;
import java.time.temporal.TemporalAccessor;
import java.time.temporal.TemporalQuery;

/**
 * Created by tao.zeng on 2020-01-03.
 */
public class QuarterCycle {


    /**
     * 定义四个季度枚举值
     */
    enum QuarterEnum {
        FIRST, SECOND, THIRD, FOURTH
    }

    class QuarterOfYearQuery implements TemporalQuery<QuarterEnum> {
        @Override
        public QuarterEnum queryFrom(TemporalAccessor temporal) {
            LocalDate now = LocalDate.from(temporal);
            if (now.isBefore(now.with(Month.APRIL).withDayOfMonth(1))) {
                return QuarterEnum.FIRST;
            } else if (now.isBefore(now.with(Month.JULY).withDayOfMonth(1))) {
                return QuarterEnum.SECOND;
            } else if (now.isBefore(now.with(Month.OCTOBER).withDayOfMonth(1))) {
                return QuarterEnum.THIRD;
            } else {
                return QuarterEnum.FOURTH;
            }
        }
    }

    /**
     * 根据 LocalDateTime 获取季度
     *
     * @param dateTime
     * @return
     */
    public int getQuarter(LocalDateTime dateTime) {
        return getQuarter(dateTime.toLocalDate());
    }

    /**
     * 根据 LocalDate 获取季度
     *
     * @param date
     * @return
     */
    public int getQuarter(LocalDate date) {
        TemporalQuery<QuarterEnum> quarterOfYearQuery = new QuarterOfYearQuery();
        QuarterEnum quarter = date.query(quarterOfYearQuery);
        switch (quarter) {
            case FIRST:
                return 1;
            case SECOND:
                return 2;
            case THIRD:
                return 3;
            case FOURTH:
                return 4;
        }
        return 0;
    }


    public Tuple2<LocalDateTime, LocalDateTime> getQuarterRange(LocalDateTime dateTime, int quarter) {
        return getQuarterRange(dateTime.getYear(), quarter);
    }

    public Tuple2<LocalDateTime, LocalDateTime> getQuarterRange(LocalDate dateTime, int quarter) {
        return getQuarterRange(dateTime.getYear(), quarter);
    }

    /**
     * 获取某年某季度的第一天和最后一天
     *
     * @param year    哪一年
     * @param quarter 第几季度
     */
    public Tuple2<LocalDateTime, LocalDateTime> getQuarterRange(int year, int quarter) {
        LocalDate startDate, endDate;
        switch (quarter) {
            case 1:
                // 01.01-03.31
                startDate = LocalDate.of(year, 1, 1);
                endDate = LocalDate.of(year, 3, 31);
                break;
            case 2:
                // 04.01-06.30
                startDate = LocalDate.of(year, 4, 1);
                endDate = LocalDate.of(year, 6, 30);
                break;
            case 3:
                // 07.01-09.30
                startDate = LocalDate.of(year, 7, 1);
                endDate = LocalDate.of(year, 9, 30);
                break;
            case 4:
                // 10.01-12.31
                startDate = LocalDate.of(year, 10, 1);
                endDate = LocalDate.of(year, 12, 31);
                break;
            default:
                throw new RuntimeException("quarter range [1-4]");
        }
        return Tuple.of(startDate.atTime(LocalTime.MIN), endDate.atTime(LocalTime.MAX));
    }
}
```



### 测试代码

```java
QuarterCycle quarterCycle = new QuarterCycle();

LocalDateTime now = LocalDateTime.now();
System.out.println("当前时间:" + now + "\n");

int quarter = quarterCycle.getQuarter(now);
System.out.println("当前季度:" + quarter + "\n");

Tuple2<LocalDateTime, LocalDateTime> tuple2 = quarterCycle.getQuarterRange(now, quarter);
System.out.println("当前季度开始和结束时间:" + tuple2 + "\n");

// 输出结果
当前时间:2020-01-04T15:17:43.949

当前季度:1

当前季度开始和结束时间:(2020-01-01T00:00, 2020-03-31T23:59:59.999999999)
```





## 常用方法整理




| 方法 | 作用 |
| ---- | ---- |
|adjustInto | 调整指定的Temporal和当前LocalDateTime对 |
| atOffset |   结合LocalDateTime和ZoneOffset创建一个 |
| atZone | 结合LocalDateTime和指定时区创建一个ZonedD |
| compareTo |  比较两个LocalDateTime |
| format | 格式化LocalDateTime生成一个字符串 |
| from |   转换TemporalAccessor为LocalDateTime |
| get |得到LocalDateTime的指定字段的值 |
| getDayOfMonth |  得到LocalDateTime是月的第几天 |
| getDayOfWeek |   得到LocalDateTime是星期几 |
| getDayOfYear |   得到LocalDateTime是年的第几天 |
| getHour |得到LocalDateTime的小时 |
| getLong |得到LocalDateTime指定字段的值 |
| getMinute |  得到LocalDateTime的分钟 |
| getMonth |   得到LocalDateTime的月份 |
| getMonthValue |  得到LocalDateTime的月份，从1到12 |
| getNano |得到LocalDateTime的纳秒数 |
| getSecond |  得到LocalDateTime的秒数 |
| getYear |得到LocalDateTime的年份 |
| isAfter |判断LocalDateTime是否在指定LocalDateTime之后 |
| isBefore |   判断LocalDateTime是否在指定LocalDateTime之前 |
| isEqual |判断两个LocalDateTime是否相等 |
| isSupported |判断LocalDateTime是否支持指定时间字段或单元 |
| minus |  返回LocalDateTime减去指定数量的时间得到的值 |
| minusDays |  返回LocalDateTime减去指定天数得到的值 |
| minusHours | 返回LocalDateTime减去指定小时数得到的值 |
| minusMinutes |   返回LocalDateTime减去指定分钟数得到的值 |
| minusMonths |返回LocalDateTime减去指定月数得到的值 |
| minusNanos | 返回LocalDateTime减去指定纳秒数得到的值 |
| minusSeconds |   返回LocalDateTime减去指定秒数得到的值 |
| minusWeeks | 返回LocalDateTime减去指定星期数得到的值 |
| minusYears | 返回LocalDateTime减去指定年数得到的值 |
| now |返回指定时钟的当前LocalDateTime |
| of | 根据年、月、日、时、分、秒、纳秒等创建LocalDateTime |
| ofEpochSecond |  根据秒数(从1970-01-0100:00:00开始)创建LocalDateTime |
| ofInstant |  根据Instant和ZoneId创建LocalDateTime |
| parse |  解析字符串得到LocalDateTime |
| plus |   返回LocalDateTime加上指定数量的时间得到的值 |
| plusDays |   返回LocalDateTime加上指定天数得到的值 |
| plusHours |  返回LocalDateTime加上指定小时数得到的值 |
| plusMinutes |返回LocalDateTime加上指定分钟数得到的值 |
| plusMonths | 返回LocalDateTime加上指定月数得到的值 |
| plusNanos |  返回LocalDateTime加上指定纳秒数得到的值 |
| plusSeconds |返回LocalDateTime加上指定秒数得到的值 |
| plusWeeks |  返回LocalDateTime加上指定星期数得到的值 |
| plusYears |  返回LocalDateTime加上指定年数得到的值 |
| query |  查询LocalDateTime |
| range |  返回指定时间字段的范围 |
| toLocalDate |返回LocalDateTime的LocalDate部分 |
| toLocalTime |返回LocalDateTime的LocalTime部分 |
| toString |   返回LocalDateTime的字符串表示 |
| truncatedTo |返回LocalDateTime截取到指定时间单位的拷贝 |
| until |  计算LocalDateTime和另一个LocalDateTime |
| with |   返回LocalDateTime指定字段更改为新值后的拷贝 |
| withDayOfMonth | 返回LocalDateTime月的第几天更改为新值后的拷贝 |
| withDayOfYear |  返回LocalDateTime年的第几天更改为新值后的拷贝 |
| withHour |   返回LocalDateTime的小时数更改为新值后的拷贝 |
| withMinute | 返回LocalDateTime的分钟数更改为新值后的拷贝 |
| withMonth |  返回LocalDateTime的月份更改为新值后的拷贝 |
| withNano |   返回LocalDateTime的纳秒数更改为新值后的拷贝 |
| withSecond | 返回LocalDateTime的秒数更改为新值后的拷贝 |
| withYear |   返回LocalDateTime年份更改为新值后的拷贝 |


