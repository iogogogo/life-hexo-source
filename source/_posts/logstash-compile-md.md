---
title: logstash修改默认时区为东八区
date: 2020-12-19 10:46:41
tags: logstash
categories:
---

上一篇文章我们讲了`logstash-filter-date`插件怎么支持纳秒并且进行编译，最后提到了一个时区问题，我们说说logstash默认的时区问题，这里主要涉及两个类

- org.logstash.StringInterpolation
- org.logstash.Timestamp



整体的思路和编译`logstash-filter-date`插件类似，主要就是修改以上两个类的时区，然后在编译即可。

- 依赖安装
- 下载源代码
- 修改源码
- 网络问题造成下载失败或者缓慢解决方案
- 构建snapshot package
- 使用snapshot package测试时区问题

## 依赖安装

- gradle

根据项目`gradle/wrapper/gradle-wrapper.properties`下载`gradle-x.x.x-bin.zip`，放在`~/.gradle/wrapper/dists/gradle-6.5.1-bin/1m5048aptkfynhbvolwgr4ej9/`

```http
https://services.gradle.org/distributions/
```

- ruby

- jruby
- rvm
- rbenv

```http
https://rubygems.org/
```



## 下载源代码

```shell
cd ~/share/tmp
# 如果嫌弃下载的慢，可以使用gitee进行导入，然后根据gitee的地址下载
git clone https://github.com/elastic/logstash

# 根据tag切换到7.9.3分支
git checkout -b v7.9.3 v7.9.3
```



## 网络问题造成下载失败或者缓慢解决方案

### jruby-complete-9.2.13.0.jar

这个包下载会比较慢，可以使用下载工具下载，让将`jruby-complete-9.2.13.0.jar`放在`~/.gradle/caches/modules-2/files-2.1/org.jruby/jruby-complete/9.2.13.0/xxx`目录下。

```http
cd ~/share/tmp
wget https://repo.maven.apache.org/maven2/org/jruby/jruby-complete/9.2.13.0/jruby-complete-9.2.13.0.jar
```

### jruby-dist-9.2.13.0-bin.tar.gz

这个包每次编译时会用到，下载也比较慢，可以使用下载工具提前下载好，然后放在`$LOGSTASH_HOME/vendor/_/`下
```http
cd ~/share/tmp
wget https://repo1.maven.org/maven2/org/jruby/jruby-dist/9.2.13.0/jruby-dist-9.2.13.0-bin.tar.gz

# 根据实际路径自行替换
cp ~/share/tmp/jruby-dist-9.2.13.0-bin.tar.gz vendor/_
```



## 修改源码

### org.logstash.StringInterpolation

该类需要修改`org.logstash.StringInterpolation#evaluate(Event, String)`方法

```java
// 修改前
builder.append(t != null ? event.getTimestamp().getTime().toString( DateTimeFormat.forPattern(template.substring(open + 3, close)) .withZone(DateTimeZone.UTC)) : "" );

// 修改后，将UTC时区改为东八区
builder.append(t != null ? event.getTimestamp().getTime().toString( DateTimeFormat.forPattern(template.substring(open + 3, close)) .withZone(DateTimeZone.forID("+08:00"))) : "" );
```





### org.logstash.Timestamp

该类需要修改一个常量，也是将UTC时区改为东八区

```java
// 修改前
private static final Chronology UTC_CHRONOLOGY = ISOChronology.getInstance(DateTimeZone.UTC);

// 修改后
private static final Chronology UTC_CHRONOLOGY = ISOChronology.getInstance(DateTimeZone.forID("+08:00"));
```





## 构建snapshot package

### Building Logstash

```shell
rake bootstrap
```



### Building Artifacts

```shell
# cd $LOGSTASH_HOME
./gradlew assembleTarDistribution
```

### 编译日志

```shell
➜  logstash git:(v7.9.3) ✗ ./gradlew assembleTarDistribution                           
To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/6.5.1/userguide/gradle_daemon.html.
Daemon will be stopped at the end of the build stopping after processing

> Task :downloadJRuby UP-TO-DATE
Download https://repo1.maven.org/maven2/org/jruby/jruby-dist/9.2.13.0/jruby-dist-9.2.13.0-bin.tar.gz

> Task :logstash-core:compileJava
注: Processing Log4j annotations
注: Annotations processed
注: Processing Log4j annotations
注: No elements to process

> Task :installBundler
Fetching bundler-1.17.3.gem
Successfully installed bundler-1.17.3
1 gem installed

> Task :assembleTarDistribution
Invoking bundler install...
Using rake 12.3.3
Using public_suffix 3.1.1
Using addressable 2.7.0
Using cabin 0.9.0
Using arr-pm 0.0.10
Using atomic 1.1.101 (java)
Using backports 3.18.2
Using builder 3.2.4
Using bundler 1.17.3
Using ffi 1.13.1 (java)
Using childprocess 0.9.0
Using numerizer 0.1.1
Using chronic_duration 0.10.6
Using clamp 0.6.5
Using coderay 1.1.3
Using concurrent-ruby 1.1.7
Using dotenv 2.7.6
Using multi_json 1.15.0
Using elasticsearch-api 5.0.5
Using multipart-post 2.1.1
Using faraday 0.15.4
Using elasticsearch-transport 5.0.5
Using elasticsearch 5.0.5
Using filesize 0.2.0
Using json 1.8.6 (java)
Using fpm 1.3.3
Using gems 1.2.0
Using i18n 1.8.5
Using insist 1.0.0
Using jrjackson 0.4.12 (java)
Using jruby-openssl 0.10.4 (java)
Using openssl_pkcs8_pure 0.0.0.2
Using manticore 0.7.0 (java)
Using minitar 0.9
Using mustermann 1.0.3
Using method_source 1.0.0
Using spoon 0.0.6
Using pry 0.13.1 (java)
Using nio4r 2.5.4 (java)
Using puma 4.3.6 (java)
Using rack 2.2.3
Using rubyzip 1.3.0
Using rack-protection 2.1.0
Using tilt 2.0.10
Using sinatra 2.1.0
Using stud 0.0.23
Using thread_safe 0.3.6 (java)
Using polyglot 0.3.5
Using treetop 1.6.11
Using logstash-core 7.9.3 (java) from source at `logstash-core`
Using logstash-core-plugin-api 2.1.16 (java) from source at `logstash-core-plugin-api`
Using logstash-mixin-ecs_compatibility_support 1.0.0 (java)
Using logstash-output-elasticsearch 10.6.2 (java)
Using mustache 0.99.8
Using sawyer 0.8.2
Using octokit 4.18.0
Using paquet 0.2.1
Using pleaserun 0.0.31
Using ruby-progressbar 1.10.1
Bundle complete! 25 Gemfile dependencies, 59 gems now installed.
Gems in the group development were not installed.
Bundled gems are installed into `./vendor/bundle`
[plugin:install-default] Installing default plugins
Installing logstash-codec-avro, logstash-codec-cef, logstash-codec-collectd, logstash-codec-dots, logstash-codec-edn, logstash-codec-edn_lines, logstash-codec-es_bulk, logstash-codec-fluent, logstash-codec-graphite, logstash-codec-json, logstash-codec-json_lines, logstash-codec-line, logstash-codec-msgpack, logstash-codec-multiline, logstash-codec-netflow, logstash-codec-plain, logstash-codec-rubydebug, logstash-filter-aggregate, logstash-filter-anonymize, logstash-filter-cidr, logstash-filter-clone, logstash-filter-csv, logstash-filter-date, logstash-filter-de_dot, logstash-filter-dissect, logstash-filter-dns, logstash-filter-drop, logstash-filter-elasticsearch, logstash-filter-fingerprint, logstash-filter-geoip, logstash-filter-grok, logstash-filter-http, logstash-filter-json, logstash-filter-kv, logstash-filter-memcached, logstash-filter-metrics, logstash-filter-mutate, logstash-filter-prune, logstash-filter-ruby, logstash-filter-sleep, logstash-filter-split, logstash-filter-syslog_pri, logstash-filter-throttle, logstash-filter-translate, logstash-filter-truncate, logstash-filter-urldecode, logstash-filter-useragent, logstash-filter-uuid, logstash-filter-xml, logstash-input-azure_event_hubs, logstash-input-beats, logstash-input-couchdb_changes, logstash-input-dead_letter_queue, logstash-input-elasticsearch, logstash-input-exec, logstash-input-file, logstash-input-ganglia, logstash-input-gelf, logstash-input-generator, logstash-input-graphite, logstash-input-heartbeat, logstash-input-http, logstash-input-http_poller, logstash-input-imap, logstash-input-jms, logstash-input-pipe, logstash-input-redis, logstash-input-s3, logstash-input-snmp, logstash-input-snmptrap, logstash-input-sqs, logstash-input-stdin, logstash-input-syslog, logstash-input-tcp, logstash-input-twitter, logstash-input-udp, logstash-input-unix, logstash-integration-jdbc, logstash-integration-kafka, logstash-integration-rabbitmq, logstash-output-cloudwatch, logstash-output-csv, logstash-output-elastic_app_search, logstash-output-elasticsearch, logstash-output-email, logstash-output-file, logstash-output-graphite, logstash-output-http, logstash-output-lumberjack, logstash-output-nagios, logstash-output-null, logstash-output-pipe, logstash-output-redis, logstash-output-s3, logstash-output-sns, logstash-output-sqs, logstash-output-stdout, logstash-output-tcp, logstash-output-udp, logstash-output-webhdfs
<============-> 98% EXECUTING [9m 27s]
> :assembleTarDistribution


```



### 编译完成目录结构

```shell

```



### 常见问题
> 打包报错 Could not find tools.jar. Please check that /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home contains a valid JDK installation.

解决方案：https://www.cnblogs.com/johnjackson/p/14040958.html





> In Gemfile:
>   logstash-filter-geoip
> Error Bundler::InstallError, retrying 8/10
> Bundler::GemspecError: Could not read gem at /Users/tao.zeng/share/workspaces/opensource/logstash/vendor/bundle/jruby/2.5.0/cache/logstash-filter-geoip-6.0.3-java.gem. It may be corrupted.
> An error occurred while installing logstash-filter-geoip (6.0.3), and Bundler cannot continue.
> Make sure that `gem install logstash-filter-geoip -v '6.0.3' --source 'https://rubygems.org/'` succeeds before bundling.

该问题是logstash-filter-geoip没有按照，需要手动执行安装，但是由于gem的源特别慢，可以使用国内的源。

参考地址：[Ruby Gems 镜像](https://developer.aliyun.com/mirror/rubygems)

执行安装命令，后面的source如果设置了全局代理，则可以省略
```shell
gem install logstash-filter-geoip -v '6.0.3' --source https://mirrors.aliyun.com/rubygems/
```

## 使用snapshot package测试时区问题



```shell
➜  logstash-7.9.3 ./lsboot conf/test.conf test
bin/logstash -f conf/test.conf -l logs/test --path.data data/test -n test
Sending Logstash logs to logs/test which is now configured via log4j2.properties
[2020-12-29T19:12:34,823][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.9.3", "jruby.version"=>"jruby 9.2.13.0 (2.5.7) 2020-08-03 9a89c94bcc Java HotSpot(TM) 64-Bit Server VM 25.201-b09 on 1.8.0_201-b09 +indy +jit [darwin-x86_64]"}
[2020-12-29T19:12:35,003][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.queue", :path=>"data/test/queue"}
[2020-12-29T19:12:35,007][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.dead_letter_queue", :path=>"data/test/dead_letter_queue"}
[2020-12-29T19:12:35,101][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2020-12-29T19:12:35,139][INFO ][logstash.agent           ] No persistent UUID file found. Generating new UUID {:uuid=>"ea54c753-bafe-4a0c-b064-5409e9fe1114", :path=>"data/test/uuid"}
[2020-12-29T19:12:37,276][INFO ][org.reflections.Reflections] Reflections took 44 ms to scan 1 urls, producing 22 keys and 45 values
[2020-12-29T19:12:38,937][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>8, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>1000, "pipeline.sources"=>["/Users/tao.zeng/share/software/logstash-7.9.3/conf/test.conf"], :thread=>"#<Thread:0x68346a58 run>"}
[2020-12-29T19:12:39,895][INFO ][logstash.javapipeline    ][main] Pipeline Java execution initialization time {"seconds"=>0.95}
[2020-12-29T19:12:39,952][INFO ][logstash.inputs.stdin    ][main] Automatically switching from json to json_lines codec {:plugin=>"stdin"}
[2020-12-29T19:12:39,999][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2020-12-29T19:12:40,062][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2020-12-29T19:12:40,310][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
[ { "type":"test", "parent": "TOTAL", "children": [ "TRANSCODE", "HOSTNAME", "MICROAPP" ] } ]
{
      "children" => [
        [0] "TRANSCODE",
        [1] "HOSTNAME",
        [2] "MICROAPP"
    ],
          "type" => "test",
        "parent" => "TOTAL",
       "unix_ts" => 1609240365076,
          "host" => "TaoZeng.MBP",
    "@timestamp" => 2020-12-29T19:12:45.076+08:00, # 默认时区已经改为东八区的时间
          "name" => "哈哈哈",
      "@version" => "1"
}

```

