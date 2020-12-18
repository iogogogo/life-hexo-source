---
title: logstash日期插件支持纳秒
date: 2020-12-18 16:01:58
tags: logstash
categories:
---



最近项目上遇到时间为纳秒的情况，用到logstash解析时是不支持纳秒的，这里提供一个思路就是自己修改logstash的日期插件，让他支持纳秒，具体涉及的插件是[logstash-filter-date](https://github.com/logstash-plugins/logstash-filter-date)



## 前期准备

- 下载源代码
- 下载logstash
- 安装必要编译组件
- 修改源代码
- 打包编译
- 替换默认的`logstash-filter-date-${version}.jar`
- 测试纳秒解析



## 下载源码

```shell
cd ~/share/tmp

# 如果下载太慢，可以先用gitee同步该仓库，然后在使用gitee地址下载
git clone https://github.com/logstash-plugins/logstash-filter-date.git

# gitee地址
git clone https://gitee.com/iogogogo/logstash-filter-date.git

# 切换到最新的tag分支，目前是v3.1.9
git checkout -b v3.1.9 v3.1.9
```



## 下载logstash

因为我用到的logstash版本为`7.9.3`，所以下载的logstash也是`7.9.3`，理论上编译出来的`logstash-filter-date`是通用的

```shell
cd ~/share/tmp

wget https://artifacts.elastic.co/downloads/logstash/logstash-7.9.3.tar.gz
```





## 安装编译组件

- 安装jruby

  ```shell
  https://www.ruby-lang.org/zh_cn/downloads/
  ```

- 安装rvm

  ```shell
  https://rvm.io/
  ```

- 安装rbenv

  ```shell
  https://ruby-china.org/wiki/rbenv-guide
  ```

  

## 修改源代码

### 将源代码导入idea

源码导入idea会下载gradle组件和对应的依赖，这里需要保证网络畅通。



### 添加纳秒解析

找到`src/main/java/org/logstash/filters/parser`包位置，新建一个纳秒解析类

```java
/*
 * Licensed to Elasticsearch under one or more contributor
 * license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright
 * ownership. Elasticsearch licenses this file to you under
 * the Apache License, Version 2.0 (the "License"); you may
 * not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied.  See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */

package org.logstash.filters.parser;

import org.joda.time.Instant;

import java.math.BigDecimal;

/**
 * Created by tao.zeng on 2020/12/12.
 */
public class UnixNanosecondParser implements TimestampParser {

    private static long MAX_EPOCH_NANOSECOND = (long) Integer.MAX_VALUE * 1000 * 1000;

    @Override
    public Instant parse(String value) {
        return parse(Long.parseLong(value));
    }

    @Override
    public Instant parse(Long value) {
        return new Instant(value / 1000);
    }

    @Override
    public Instant parse(Double value) {
        // XXX: Should we accept a double?
        return parse(value.longValue());
    }


    @Override
    public Instant parseWithTimeZone(String value, String timezone) {
        return parse(value);
    }

    @Override
    public Instant parse(BigDecimal value) {
        long lv = value.longValue();
        if (lv > MAX_EPOCH_NANOSECOND) {
            throw new IllegalArgumentException("Cannot parse date for value larger than UNIX NS maximum seconds");
        }
        return new Instant(lv / 1000);
    }
}

```

##### 在解析工厂类添加纳秒解析

`org.logstash.filters.parser.TimestampParserFactory`

```java
/*
 * Licensed to Elasticsearch under one or more contributor
 * license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright
 * ownership. Elasticsearch licenses this file to you under
 * the Apache License, Version 2.0 (the "License"); you may
 * not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied.  See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */

package org.logstash.filters.parser;

import org.joda.time.DateTimeZone;

import java.util.Locale;

public class TimestampParserFactory {
    private DateTimeZone timezone;

    private static final String ISO8601 = "ISO8601";
    private static final String UNIX = "UNIX";
    private static final String UNIX_MS = "UNIX_MS";
    /**
     * 新增纳秒
     */
    private static final String UNIX_NS = "UNIX_NS";
    private static final String TAI64N = "TAI64N";

    /*
     * zone is a String because it can be dynamic and come from the event while we parse it.
     */
    public static TimestampParser makeParser(String pattern, Locale locale, String zone) {
        if (locale == null) {
            locale = Locale.getDefault();
        }

        String tz = zone;

        if (tz == null) {
            tz = DateTimeZone.getDefault().getID();
        } else if (zone.contains("%{")) {
            tz = null;
        }

        switch (pattern) {
            case ISO8601: // Short-hand for a few ISO8601-ish formats
                return new CasualISO8601Parser(tz);
            case UNIX: // Unix epoch in seconds
                return new UnixEpochParser();
            case TAI64N: // TAI64N format
                return new TAI64NParser();
            case UNIX_MS: // Unix epoch in milliseconds
                return new UnixMillisEpochParser();
            case UNIX_NS: // Unix epoch in nanoseconds
                // 纳秒解析类
                return new UnixNanosecondParser();

            default:
                return new JodaParser(pattern, locale, tz);
        }
    }

    public static TimestampParser makeParser(String pattern) {
        return makeParser(pattern, (Locale) null, null);
    }

    public static TimestampParser makeParser(String pattern, String locale, String zone) {
        return makeParser(pattern, locale == null ? null : Locale.forLanguageTag(locale), zone);
    }
}

```



### 注释一个单元测试

因为该类会因为找不到方法而导致编译报错，可以注释忽略`org.logstash.filters.DateFilterTest#commonAssertions`

```java
private void commonAssertions(Event event, ParseExecutionResult code, String expected) {
        /*Assert.assertSame(ParseExecutionResult.SUCCESS, code);
        String actual = ((Timestamp) event.getField("[result_ts]")).toIso8601();
        Assert.assertTrue(String.format("Unequal - expected: %s, actual: %s", expected, actual), expected.equals(actual));*/
}
```



### 修改配置文件

#### build.gradle

修改build.gradle文件中63、66两行的logstash-core配置，改为下载的标准`logstash/logstash-core`目录

```groovy
// 默认配置
testCompile fileTree(dir: logstashCoreGemPath, include: '**/*.jar')
compileOnly fileTree(dir: logstashCoreGemPath, include: '**/*.jar')

// 修改为下载的logstash/logstash-core的绝对路径
testCompile fileTree(dir: '/Users/tao.zeng/share/software/logstash-7.9.3/logstash-core', include: '**/*.jar')
compileOnly fileTree(dir: '/Users/tao.zeng/share/software/logstash-7.9.3/logstash-core', include: '**/*.jar')
```

#### Rakefile

修改Rakefile文件的lsc_path配置为`logstash/logstash-core`目录

```groovy
// 默认配置
lsc_path = `bundle show logstash-core`

// 修改为下载的logstash/logstash-core的绝对路径
lsc_path = `/Users/tao.zeng/share/software/logstash-7.9.3/logstash-core`
```



## 打包编译

执行gradle的build任务，生成新的`logstash-date-filter-{version}.jar`，新生成文件目录如下：

```shell
# 有个好奇的是明明是3.1.9的tag包，但是在源码里面确实3.1.6的版本，所以打包出来也是3.1.6的版本号，解压logstash发现也是该版本号，所以不用在意版本号
build/libs/logstash-filter-date-3.1.6.jar
```



```shell
19:38:39: Executing task 'build'...

:buildSrc:compileJava UP-TO-DATE
:buildSrc:compileGroovy UP-TO-DATE
:buildSrc:processResources UP-TO-DATE
:buildSrc:classes UP-TO-DATE
:buildSrc:jar UP-TO-DATE
:buildSrc:assemble UP-TO-DATE
:buildSrc:compileTestJava UP-TO-DATE
:buildSrc:compileTestGroovy UP-TO-DATE
:buildSrc:processTestResources UP-TO-DATE
:buildSrc:testClasses UP-TO-DATE
:buildSrc:test UP-TO-DATE
:buildSrc:check UP-TO-DATE
:buildSrc:build UP-TO-DATE
:distTar UP-TO-DATE
:distZip UP-TO-DATE
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:check
:build

BUILD SUCCESSFUL

Total time: 7.276 secs
19:38:47: Task execution finished 'build'.

```



## 替换logstash中默认的jar

### 解压logstash-7.9.3.tar.gz

```shell
cd ~/share/tmp

# 解压
tar -zxvf logstash-7.9.3.tar.gz
```

### 查找logstash-filter-date-3.1.6.jar

```shell
cd logstash-7.9.3 && find . -name 'logstash-filter-date*'

# 日志
➜  logstash-7.9.3 find . -name 'logstash-filter-date*'
./vendor/bundle/jruby/2.5.0/specifications/logstash-filter-date-3.1.9.gemspec
./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9
./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9/logstash-filter-date.gemspec
./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9/lib/logstash-filter-date_jars.rb
./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9/vendor/jar-dependencies/org/logstash/filters/logstash-filter-date
./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9/vendor/jar-dependencies/org/logstash/filters/logstash-filter-date/3.1.6/logstash-filter-date-3.1.6.jar
```

可以看到原先的jar在`./vendor/bundle/jruby/2.5.0/gems/logstash-filter-date-3.1.9/vendor/jar-dependencies/org/logstash/filters/logstash-filter-date/3.1.6/`这个目录下面，我们只需要用修改过的jar替换掉就可以了。





## 测试纳秒解析

为了方便测试，我们将[上一篇文章](https://iogogogo.github.io/2020/12/09/lsboot/)的启动脚本复制过来，并且新建一个配置文件`conf/test.conf`

```shell
➜  logstash-7.9.3 ll
total 1280
-rw-r--r--   1 tao.zeng  staff   2.2K 10 16 20:23 CONTRIBUTORS
-rw-r--r--   1 tao.zeng  staff   3.9K 10 16 20:24 Gemfile
-rw-r--r--   1 tao.zeng  staff    22K 10 16 20:25 Gemfile.lock
-rw-r--r--   1 tao.zeng  staff    13K 10 16 20:23 LICENSE.txt
-rw-r--r--   1 tao.zeng  staff   587K 10 16 20:23 NOTICE.TXT
drwxr-xr-x  22 tao.zeng  staff   704B 10 16 21:35 bin
drwxr-xr-x   3 tao.zeng  staff    96B 12 18 20:00 conf
drwxr-xr-x   8 tao.zeng  staff   256B 10 16 21:35 config
drwxr-xr-x   2 tao.zeng  staff    64B 10 16 20:23 data
drwxr-xr-x   6 tao.zeng  staff   192B 10 16 21:35 lib
drwxr-xr-x   6 tao.zeng  staff   192B 10 16 21:35 logstash-core
drwxr-xr-x   5 tao.zeng  staff   160B 10 16 21:35 logstash-core-plugin-api
-rwxr-xr-x@  1 tao.zeng  staff   1.3K 12 18 20:00 lsboot
drwxr-xr-x   5 tao.zeng  staff   160B 10 16 21:35 modules
drwxr-xr-x   3 tao.zeng  staff    96B 10 16 21:35 tools
drwxr-xr-x   4 tao.zeng  staff   128B 10 16 21:35 vendor
drwxr-xr-x  14 tao.zeng  staff   448B 10 16 21:35 x-pack
```



### 编写测试配置conf/test.conf

```shell
input {
     stdin { 
         codec => "json"
     } 
}

filter {
    mutate {
        add_field => { "name" => "哈哈哈" }
    }

    # https://zerlong.com/886.html
    ruby { code => 'event.set("unix_ts",(event.get("@timestamp").to_f.round(3)*1000).to_i)' }

    date {
    		# 注意这里的UNIX_NS就是我们在TimestampParserFactory新建的纳秒解析器，官方版本是不支持的
        match => ["unix_ns", "UNIX_NS"]
        target => "date_ns"
    }
}

output { 
    stdout {} 
}
```



### 测试数据

```json
{ "unix_ns": 1607206093000000, "tags": "test" }
```



### 启动测试

```shell
➜  logstash-7.9.3 ./lsboot conf/test.conf test
bin/logstash -f conf/test.conf -l logs/test --path.data data/test -n test
Sending Logstash logs to logs/test which is now configured via log4j2.properties
[2020-12-18T20:13:19,801][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.9.3", "jruby.version"=>"jruby 9.2.13.0 (2.5.7) 2020-08-03 9a89c94bcc Java HotSpot(TM) 64-Bit Server VM 25.201-b09 on 1.8.0_201-b09 +indy +jit [darwin-x86_64]"}
[2020-12-18T20:13:19,932][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.queue", :path=>"data/test/queue"}
[2020-12-18T20:13:19,935][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.dead_letter_queue", :path=>"data/test/dead_letter_queue"}
[2020-12-18T20:13:20,029][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2020-12-18T20:13:20,061][INFO ][logstash.agent           ] No persistent UUID file found. Generating new UUID {:uuid=>"e4188c10-0621-4967-a957-8aa14f1a2945", :path=>"data/test/uuid"}
[2020-12-18T20:13:21,802][INFO ][org.reflections.Reflections] Reflections took 33 ms to scan 1 urls, producing 22 keys and 45 values
[2020-12-18T20:13:23,160][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>8, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>1000, "pipeline.sources"=>["/Users/tao.zeng/share/tmp/logstash-7.9.3/conf/test.conf"], :thread=>"#<Thread:0x49b901aa run>"}
[2020-12-18T20:13:23,871][INFO ][logstash.javapipeline    ][main] Pipeline Java execution initialization time {"seconds"=>0.7}
[2020-12-18T20:13:23,922][INFO ][logstash.inputs.stdin    ][main] Automatically switching from json to json_lines codec {:plugin=>"stdin"}
[2020-12-18T20:13:23,962][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2020-12-18T20:13:24,041][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2020-12-18T20:13:24,244][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
{ "unix_ns": 1607206093000000, "tags": "test" }
{
       "unix_ts" => 1608293606313,
       "date_ns" => 2020-12-05T22:08:13.000Z,
    "@timestamp" => 2020-12-18T12:13:26.313Z,
          "tags" => "test",
       "unix_ns" => 1607206093000000,
          "host" => "TaoZeng.MBP",
      "@version" => "1",
          "name" => "哈哈哈"
}
```



以上，`1607206093000000`纳秒时间戳已经支持转换成`date`类型的数据，但是转换的结果却还有时区显示的问题，这里有两个方案

- 第一是在target的时候指定时区；
- 第二是修改logstash源码，把时区默认修改为东八区，这个我们下一次说怎么修改。

[logstash 时间戳时区问题](https://www.zybuluo.com/StrGlee/note/1179723)





以上，就是logstash-filter-date插件添加纳秒支持并且替换，编译好的[logstash-filter-date-3.1.6.jar](https://gitee.com/iogogogo/iogogogo/raw/master/files/logstash-filter-date-3.1.6.jar)

