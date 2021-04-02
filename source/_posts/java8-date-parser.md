---
title: 使用Java8实现常用日期解析封装
date: 2021-04-02 21:06:32
tags: Java
categories: 
---

项目上做ELT数据清洗的时候，经常会遇到各种各样的日期格式，使用logstash解析日期是可以指定多个pattern，logstash会按照顺序按个去匹配，直到匹配上则返回，自己写数据处理的时候，也有类似的需求，看了下logstash关于日期解析处理的源码，主要是使用`joda-time`这个库完成的，在Java8以后，也有类似的日期处理api，下面使用Java8自带的api实现一个简单的封装，整体思路和logstash处理日期的是一样的，通过工厂类，创建出不同的解析器，通过具体的解析器针对pattern进行匹配。

没有什么太复杂的业务逻辑，直接看代码，代码注释也比较全面。



代码地址入口：[Java8DateTimeUtils](https://gitee.com/iogogogo/iogogogo-parent/tree/master/iogogogo-common/src/main/java/com/github/iogogogo/common/util)

## Java8DateTimeUtils

```java
package com.github.iogogogo.common.util;

import com.github.iogogogo.common.util.parser.TimestampParser;
import com.github.iogogogo.common.util.parser.TimestampParserFactory;
import com.github.iogogogo.common.util.parser.impl.UnixEpochParser;
import com.github.iogogogo.common.util.parser.impl.UnixMillisEpochParser;
import io.vavr.Tuple;
import io.vavr.Tuple2;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.lang3.StringUtils;

import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.Date;
import java.util.List;
import java.util.Locale;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * The type Java 8 date time utils.
 */
@Slf4j
public class Java8DateTimeUtils {

    /**
     * Now date time local date time.
     *
     * @return the local date time
     */
    public static LocalDateTime nowDateTime() {
        return LocalDateTime.now();
    }

    /**
     * Now date local date.
     *
     * @return the local date
     */
    public static LocalDate nowDate() {
        return LocalDate.now();
    }

    /**
     * Try parse local date time.
     *
     * @param value   the value
     * @param locale  the locale
     * @param pattern the pattern
     * @return the local date time
     */
    public static LocalDateTime tryParse(String value, Locale locale, List<String> pattern) {

        if (StringUtils.isEmpty(value)) throw new NullPointerException("value is not null.");

        TimestampParser timestampParser = TimestampParserFactory.makeParser(value);

        if (timestampParser instanceof UnixEpochParser || timestampParser instanceof UnixMillisEpochParser) {
            return timestampParser.parse(null);
        } else {
            boolean flag;
            List<String> patterns = pattern.stream().filter(StringUtils::isNotEmpty).collect(Collectors.toList());
            List<String> collect = (flag = CollectionUtils.isNotEmpty(patterns)) ? patterns : TimestampParser.PATTERN_LIST;

            List<DateTimeFormatter> timeFormatters = collect.stream()
                    .peek(x -> {
                        if (log.isDebugEnabled())
                            log.debug("init {} parser ofPattern: {}", flag ? "customer" : "system", x);
                    }).map(x -> DateTimeFormatter.ofPattern(x, locale)).collect(Collectors.toList());

            return timestampParser.parse(timeFormatters);
        }
    }

    /**
     * Try parse local date time.
     *
     * @param value   the value
     * @param locale  the locale
     * @param pattern the pattern
     * @return the local date time
     */
    public static LocalDateTime tryParse(String value, Locale locale, String... pattern) {
        return tryParse(value, locale, Stream.of(pattern).collect(Collectors.toList()));
    }

    /**
     * Try parse local date time.
     *
     * @param value   the value
     * @param pattern the pattern
     * @return the local date time
     */
    public static LocalDateTime tryParse(String value, List<String> pattern) {
        return tryParse(value, Locale.CHINA, pattern);
    }

    /**
     * Try parse local date time.
     *
     * @param value   the value
     * @param pattern the pattern
     * @return the local date time
     */
    public static LocalDateTime tryParse(String value, String... pattern) {
        return tryParse(value, Stream.of(pattern).collect(Collectors.toList()));
    }


    /**
     * To instant instant.
     *
     * @param localDateTime the local date time
     * @return the instant
     */
    public static Instant toInstant(LocalDateTime localDateTime) {
        ZoneId zone = ZoneId.systemDefault();
        return localDateTime.atZone(zone).toInstant();
    }

    /**
     * To epoch milli long.
     *
     * @param localDateTime the local date time
     * @return the long
     */
    public static long toEpochMilli(LocalDateTime localDateTime) {
        return toInstant(localDateTime).toEpochMilli();
    }

    /**
     * To epoch second long.
     *
     * @param localDateTime the local date time
     * @return the long
     */
    public static long toEpochSecond(LocalDateTime localDateTime) {
        return toInstant(localDateTime).toEpochMilli() / 1000;
    }

    /**
     * Of epoch milli instant.
     *
     * @param millis the millis
     * @return the instant
     */
    public static Instant ofEpochMilli(long millis) {
        return Instant.ofEpochMilli(millis);
    }

    /**
     * Of epoch second instant.
     *
     * @param second the second
     * @return the instant
     */
    public static Instant ofEpochSecond(long second) {
        return Instant.ofEpochSecond(second);
    }

    /**
     * To java 8 millis local date time.
     *
     * @param timestamp the timestamp
     * @return the local date time
     */
    public static LocalDateTime toJava8Millis(long timestamp) {
        return toJava8(ofEpochMilli(timestamp));
    }

    /**
     * To java 8 second local date time.
     *
     * @param timestamp the timestamp
     * @return the local date time
     */
    public static LocalDateTime toJava8Second(long timestamp) {
        return toJava8(ofEpochSecond(timestamp * 1000));
    }

    /**
     * To java 8 local date time.
     *
     * @param instant the instant
     * @return the local date time
     */
    public static LocalDateTime toJava8(Instant instant) {
        ZoneId zone = ZoneId.systemDefault();
        return LocalDateTime.ofInstant(instant, zone);
    }

    /**
     * To java 8 local date time.
     *
     * @param date the date
     * @return the local date time
     */
    public static LocalDateTime toJava8(Date date) {
        return toJava8(date.toInstant());
    }

    /**
     * Millis convert tuple 2.
     *
     * @param value the value
     * @param unit  the unit
     * @return the tuple 2
     */
    public static Tuple2<Long, String> millisConvert(long value, String unit) {
        return convert(value, unit, "MS");
    }

    /**
     * Convert tuple 2.
     *
     * @param value      the value
     * @param srcUnit    the src unit
     * @param targetUnit the target unit
     * @return the tuple 2
     */
    public static Tuple2<Long, String> convert(long value, String srcUnit, String targetUnit) {
        long result;
        TimeUnit timeUnit = getUnit(targetUnit);
        switch (srcUnit.toUpperCase()) {
            case "SECONDS":
                result = TimeUnit.SECONDS.convert(value, timeUnit);
                return Tuple.of(result, String.format("%d%s", result, "秒"));
            case "MINUTES":
                result = TimeUnit.MINUTES.convert(value, timeUnit);
                return Tuple.of(result, String.format("%d%s", result, "分钟"));
            case "HOURS":
                result = TimeUnit.HOURS.convert(value, timeUnit);
                return Tuple.of(result, String.format("%d%s", result, "小时"));
            case "DAYS":
                result = TimeUnit.DAYS.convert(value, timeUnit);
                return Tuple.of(result, String.format("%d%s", result, "天"));
            case "MS":
            case "MILLIS":
            case "MILLISECONDS":
            default:
                result = value;
                return Tuple.of(result, String.format("%d%s", result, "毫秒"));
        }
    }

    /**
     * To millis long.
     *
     * @param value the value
     * @param unit  the unit
     * @return the long
     */
    public static Long toMillis(long value, String unit) {
        switch (unit.toUpperCase()) {
            case "SECONDS":
                return TimeUnit.SECONDS.toMillis(value);
            case "MINUTES":
                return TimeUnit.MINUTES.toMillis(value);
            case "HOURS":
                return TimeUnit.HOURS.toMillis(value);
            case "DAYS":
                return TimeUnit.DAYS.toMillis(value);
            case "MS":
            case "MILLIS":
            case "MILLISECONDS":
            default:
                return value;
        }
    }

    /**
     * Gets unit.
     *
     * @param unit the unit
     * @return the unit
     */
    public static TimeUnit getUnit(String unit) {
        switch (unit.toUpperCase()) {
            case "SECONDS":
                return TimeUnit.SECONDS;
            case "MINUTES":
                return TimeUnit.MINUTES;
            case "HOURS":
                return TimeUnit.HOURS;
            case "DAYS":
                return TimeUnit.DAYS;
            case "MS":
            case "MILLIS":
            case "MILLISECONDS":
            default:
                return TimeUnit.MILLISECONDS;
        }
    }
}
```

## TimestampParserFactory

```java
package com.github.iogogogo.common.util.parser;

import com.github.iogogogo.common.util.parser.impl.LocalDateTimeParser;
import com.github.iogogogo.common.util.parser.impl.UnixEpochParser;
import com.github.iogogogo.common.util.parser.impl.UnixMillisEpochParser;
import org.apache.commons.lang3.math.NumberUtils;

/**
 * Created by tao.zeng on 2021/3/26.
 */
public class TimestampParserFactory {

    private final static int UNIX_LENGTH = 10, UNIX_MS_LENGTH = 13;

    /**
     * Make parser timestamp parser.
     *
     * @param value the value
     * @return the timestamp parser
     */
    public static TimestampParser makeParser(String value) {
        if (NumberUtils.isCreatable(value)) {
            long longValue = NumberUtils.createNumber(value).longValue();
            int length = Long.toString(longValue).length();

            if (length == UNIX_LENGTH) {
                return new UnixEpochParser(longValue);
            } else if (length == UNIX_MS_LENGTH) {
                return new UnixMillisEpochParser(longValue);
            }
        }
        return new LocalDateTimeParser(value);
    }
}

```

## TimestampParser

```java
package com.github.iogogogo.common.util.parser;

import com.fasterxml.jackson.core.type.TypeReference;
import com.google.common.collect.Lists;
import com.github.iogogogo.common.util.JsonParse;
import org.apache.commons.lang3.StringUtils;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

/**
 * Created by tao.zeng on 2021/3/26.
 */
public interface TimestampParser {

    /**
     * The constant PATTERN_LIST.
     */
    List<String> PATTERN_LIST = StringUtils.isNotEmpty(System.getProperty("iogogogo.ts.parse.pattern", "")) ?
            JsonParse.tryParse(System.getProperty("iogogogo.ts.parse.pattern", ""), new TypeReference<List<String>>() {
            }) :
            defaultPattern();

    /**
     * Default pattern list.
     *
     * @return the list
     */
    static List<String> defaultPattern() {
        return Lists.newArrayList(
                "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", // 2020-08-13T08:59:59.904Z
                "yyyy-MM-dd HH:mm:ss:SSSSSS", // 2020-08-13 08:59:59:904001
                "yyyy-MM-dd'T'HH:mm:ss.SSSXXX", // 2020-08-13T08:59:59.904+08:00
                "yyyy-MM-dd HH:mm:ss:SSSSSS", // 2020-08-13 08:59:59:904001
                "yyyy-MM-dd HH:mm:ss", // 2020-08-13 08:59:59
                "yyyy-MM-dd HH:mm:ss.SSSZ",
                "yyyy-MM-dd HH:mm:ss.SSS",
                "yyyy-MM-dd HH:mm:ss,SSSZ",
                "yyyy-MM-dd HH:mm:ss,SSS"
        );
    }

    /**
     * Parse local date time.
     *
     * @param pattern the pattern
     * @return the local date time
     */
    LocalDateTime parse(List<DateTimeFormatter> pattern);
}

```



### LocalDateTimeParser

```java
package com.github.iogogogo.common.util.parser.impl;

import com.github.iogogogo.common.util.parser.TimestampParser;
import lombok.extern.slf4j.Slf4j;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.List;
import java.util.Objects;

/**
 * Created by tao.zeng on 2021/3/29.
 */
@Slf4j
public class LocalDateTimeParser implements TimestampParser {

    private final String value;

    public LocalDateTimeParser(String value) {
        this.value = value;
    }

    @Override
    public LocalDateTime parse(List<DateTimeFormatter> pattern) {
        DateTimeParseException lastException = null;
        LocalDateTime dateTime = null;
        try {
            dateTime = LocalDateTime.parse(value);
        } catch (DateTimeParseException e) {
            lastException = e;
            for (DateTimeFormatter formatter : pattern) {
                try {
                    dateTime = LocalDateTime.parse(value, formatter);
                    log.info("match formatter:{}", formatter);
                    break;
                } catch (DateTimeParseException ex) {
                    ex.addSuppressed(e);
                    lastException = ex;
                }
            }
        }

        if (Objects.nonNull(dateTime)) return dateTime;

        throw lastException;
    }

}
```



### UnixEpochParser

```java

package com.github.iogogogo.common.util.parser.impl;

import com.github.iogogogo.common.util.Java8DateTimeUtils;
import com.github.iogogogo.common.util.parser.TimestampParser;

import java.time.Instant;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

/**
 * Created by tao.zeng on 2021/3/29.
 */
public class UnixEpochParser implements TimestampParser {

    private final static long MAX_EPOCH_SECONDS = Integer.MAX_VALUE;

    private final long value;

    public UnixEpochParser(long value) {
        this.value = value;
    }

    @Override
    public LocalDateTime parse(List<DateTimeFormatter> pattern) {
        if (value > MAX_EPOCH_SECONDS) {
            throw new IllegalArgumentException("Cannot parse date for value larger than UNIX epoch maximum seconds");
        }
        Instant instant = Java8DateTimeUtils.ofEpochSecond(value);
        return Java8DateTimeUtils.toJava8(instant);
    }
}
```



### UnixMillisEpochParser

```java

package com.github.iogogogo.common.util.parser.impl;

import com.github.iogogogo.common.util.Java8DateTimeUtils;
import com.github.iogogogo.common.util.parser.TimestampParser;

import java.time.Instant;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;

/**
 * Created by tao.zeng on 2021/3/29.
 */
public class UnixMillisEpochParser implements TimestampParser {

    private final static long MAX_EPOCH_MILLISECONDS = (long) Integer.MAX_VALUE * 1000;

    private final long value;

    public UnixMillisEpochParser(long value) {
        this.value = value;
    }

    @Override
    public LocalDateTime parse(List<DateTimeFormatter> pattern) {
        if (value > MAX_EPOCH_MILLISECONDS) {
            throw new IllegalArgumentException("Cannot parse date for value larger than UNIX epoch maximum seconds");
        }
        Instant instant = Java8DateTimeUtils.ofEpochMilli(value);
        return Java8DateTimeUtils.toJava8(instant);
    }
}
```






## 单元测试
```java
package com.github.iogogogo;

import com.github.iogogogo.common.util.Java8DateTimeUtils;
import org.junit.Assert;
import org.junit.Test;

import java.time.LocalDateTime;
import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * Created by tao.zeng on 2021/4/2.
 */
public class DateTimeTests {

    @Test
    public void test() {
        String date = "2020-08-13T08:59:59.904Z";
        LocalDateTime dateTime = Java8DateTimeUtils.tryParse(date);
        Assert.assertEquals("2020-08-13T08:59:59.904", dateTime.toString());

        date = "2020-08-13 08:59:59:904001";
        dateTime = Java8DateTimeUtils.tryParse(date);
        Assert.assertEquals("2020-08-13T08:59:59.904001", dateTime.toString());

        date = "2020-08-13T08:59:59.904+08:00";
        dateTime = Java8DateTimeUtils.tryParse(date);
        Assert.assertEquals("2020-08-13T08:59:59.904", dateTime.toString());

        date = "2020-08-13 08:59:59";
        dateTime = Java8DateTimeUtils.tryParse(date);
        Assert.assertEquals("2020-08-13T08:59:59", dateTime.toString());

        dateTime = LocalDateTime.of(2020, 8, 13, 0, 0);
        long milli = Java8DateTimeUtils.toEpochMilli(dateTime);
        Assert.assertEquals(Java8DateTimeUtils.toJava8(new Date(milli)), dateTime);


        dateTime = Java8DateTimeUtils.tryParse(String.valueOf(milli));
        Assert.assertEquals("2020-08-13T00:00", dateTime.toString());


        /* ********** 自定义pattern解析********** */
        date = "2021/08/13 08:59:59";
        dateTime = Java8DateTimeUtils.tryParse(date, "yyyy/MM/dd HH:mm:ss");
        Assert.assertEquals("2021-08-13T08:59:59", dateTime.toString());

        date = "2021/08/13 08-59-59";
        dateTime = Java8DateTimeUtils.tryParse(date, "yyyy/MM/dd HH:mm:ss", "yyyy/MM/dd HH-mm-ss");
        Assert.assertEquals("2021-08-13T08:59:59", dateTime.toString());

        date = "2021/08/13 08.59.59";
        dateTime = Java8DateTimeUtils.tryParse(date, "yyyy/MM/dd HH:mm:ss", "yyyy/MM/dd HH-mm-ss", "yyyy/MM/dd HH.mm.ss");
        Assert.assertEquals("2021-08-13T08:59:59", dateTime.toString());


        Assert.assertEquals(Java8DateTimeUtils.getUnit(""), TimeUnit.MILLISECONDS);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("MS"), TimeUnit.MILLISECONDS);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("MILLIS"), TimeUnit.MILLISECONDS);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("MILLISECONDS"), TimeUnit.MILLISECONDS);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("MINUTES"), TimeUnit.MINUTES);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("HOURS"), TimeUnit.HOURS);
        Assert.assertEquals(Java8DateTimeUtils.getUnit("DAYS"), TimeUnit.DAYS);

        long i = 2 * 60 * 1000L;
        Assert.assertEquals(120000L, (long) Java8DateTimeUtils.millisConvert(i, TimeUnit.MILLISECONDS.name())._1);
        Assert.assertEquals(120, (long) Java8DateTimeUtils.millisConvert(i, TimeUnit.SECONDS.name())._1);
        Assert.assertEquals(2, (long) Java8DateTimeUtils.millisConvert(i, TimeUnit.MINUTES.name())._1);
        Assert.assertEquals(0, (long) Java8DateTimeUtils.millisConvert(i, TimeUnit.DAYS.name())._1);

        i = 2;
        Assert.assertEquals(2, (long) Java8DateTimeUtils.toMillis(i, TimeUnit.MILLISECONDS.name()));
        Assert.assertEquals(2000, (long) Java8DateTimeUtils.toMillis(i, TimeUnit.SECONDS.name()));
        Assert.assertEquals(120000, (long) Java8DateTimeUtils.toMillis(i, TimeUnit.MINUTES.name()));
        Assert.assertEquals(172800000, (long) Java8DateTimeUtils.toMillis(i, TimeUnit.DAYS.name()));
    }

}
```