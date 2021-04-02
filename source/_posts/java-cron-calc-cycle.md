---
title: 使用Java计算cron最近执行周期
date: 2021-04-02 09:32:27
tags: cron
categories: Java
---

项目中使用cron表达式来作为定时调度的参数，又需要根据执行的周期来计算一些平均数据，所以需要根据cron表达式获取执行周期。

- quartz 可以实现获取最近几次执行周期，但是如果没有quartz依赖又需要引入不必要依赖。

- spring 5.3 以上提供了 CronExpression，可以用来实现该需求。

# 使用quartz获取最近执行的时间

```java

/**
 * <dependency>
 *    <groupId>org.quartz-scheduler</groupId>
 *    <artifactId>quartz</artifactId>
 * </dependency>
 *
 */

import org.quartz.TriggerUtils;
import org.quartz.impl.triggers.CronTriggerImpl;
import org.quartz.spi.OperableTrigger;

/**
 * Latest list.
 * <p>
 * String cron = "0 0/2 * * * ?";
 *
 * @param cron  the cron
 * @param count the count
 * @return the list
 * @throws ParseException the parse exception
 */
public static List<LocalDateTime> latest(String cron, int count) throws ParseException {
    CronTriggerImpl cronTriggerImpl = new CronTriggerImpl();
    // 设置cron表达式
    cronTriggerImpl.setCronExpression(cron);
    // 根据count获取最近执行时间
    List<Date> dateList = TriggerUtils.computeFireTimes(cronTriggerImpl, null, count);
    return dateList.stream().map(Java8DateTimeUtils::toJava8).collect(Collectors.toList());
}
```

# 使用spring-context获取最近执行时间

## 获取cron当前次执行时间和前后次执行时间
```java
import org.springframework.scheduling.support.CronExpression;
import java.time.Duration;
import java.time.LocalDateTime;

/**
 * Nearby local date time.
 * <p>
 * 获取cron当前次执行时间和前后次执行时间
 * <p>
 * String cron = "0 0/2 * * * ?";
 * 
 * @param cron the cron
 * @return local date time
 */
public static Tuple3<LocalDateTime, LocalDateTime, LocalDateTime> nearby(String cron) {
    // 转换成cron表达式
    CronExpression cronExpression = CronExpression.parse(cron);
    // 下一次执行时间
    LocalDateTime next = cronExpression.next(LocalDateTime.now());

    Objects.requireNonNull(next);

    // 两次间隔时间
    long between = Duration.between(next, cronExpression.next(next)).getSeconds();

    // 当前次执行时间
    LocalDateTime current = next.minusSeconds(between);

    // 上一次执行时间
    LocalDateTime previous = current.minusSeconds(between);

    log.info("now:{} previous:{} current:{} next:{}", Java8DateTimeUtils.nowDateTime(), previous, current, next);

    return Tuple.of(previous, current, next);
}
```



## 获取最近几次的执行时间

```java

/**
 * Nearby list.
 * <p>
 * 按照count获取最近几次执行时间
 *
 * @param cron    the cron
 * @param isAfter the is after
 * @param count   the count
 * @return the list
 */
public static List<LocalDateTime> nearby(String cron, boolean isAfter, int count) {
        // 转换成cron表达式
        CronExpression cronExpression = CronExpression.parse(cron);
        // 下一次预计执行时间
        LocalDateTime nextFirst = cronExpression.next(LocalDateTime.now());

        Objects.requireNonNull(nextFirst);
        // 下下次预计执行时间
        LocalDateTime nextSecond = cronExpression.next(nextFirst);
        // 两次执行间隔
        long between = ChronoUnit.SECONDS.between(nextFirst, nextSecond);

        List<LocalDateTime> dateTimes = Lists.newArrayList();
        int i = 0;
        while (i++ < count) {
            Objects.requireNonNull(nextSecond);
            nextFirst = isAfter ? nextFirst.plusSeconds(between) : nextFirst.minusSeconds(between);
            dateTimes.add(nextFirst);
        }
        return dateTimes;
}
```





# 【参考】java通过cron表达式来获取执行的周期

原文链接：https://www.codeleading.com/article/42705195442/#_17



## 1、计算固定的周期

通过下次执行时间和下下次执行时间之差计算

```java
/**
* 根据cron表达式获取执行周期
*/
@Test
public void getPeriodByCron() {
        //30s执行一次
        String cron = "0/30 * * * * ?";
        //spring @since 5.3
        CronExpression cronExpression = CronExpression.parse(cron);
        //下次预计的执行时间
        LocalDateTime nextFirst = cronExpression.next(LocalDateTime.now());
        //下下次预计的执行时间
        LocalDateTime nextSecond = cronExpression.next(nextFirst);
        //计算周期1
        long between1 = ChronoUnit.SECONDS.between(nextFirst, nextSecond);
        Assert.assertEquals(between1, 30);
        //计算周期2
        long between2 = Duration.between(nextFirst, nextSecond).getSeconds();
        Assert.assertEquals(between2, 30);
}
```

## 2、计算多个周期

不固定的周期，获取最近几次的预定执行时间

```java
@Test
public void getEveryPeriodByCron() {
        List<Long> periods = Lists.newArrayList();
        //每小时的前5分钟每分钟执行一次，这种周期不是固定的
        String cron = "0 0,1,2,3,4,5 * * * ? ";
        //spring @since 5.3
        CronExpression cronExpression = CronExpression.parse(cron);
        //下次预计的执行时间
        LocalDateTime prevTime = cronExpression.next(LocalDateTime.now());
        int i = 0;
        while (i++ < 10) {
            Objects.requireNonNull(prevTime);
            LocalDateTime nextTime = cronExpression.next(prevTime);
            long between = ChronoUnit.SECONDS.between(prevTime, nextTime);
            prevTime = nextTime;
            periods.add(between);
        }
        System.out.println(ArrayUtils.toString(periods));
}
```