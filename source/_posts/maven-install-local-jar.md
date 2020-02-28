---
title: Maven加载本地jar文件
date: 2020-02-28 10:40:52
tags:
	- maven
categories: maven
---



日常开发中都是maven加载在远程仓库的jar文件，如果远程仓库没有相应的jar文件，一般做法就是自己传到一个特定的nexus服务器上，但是本地开发测试的时候可能nexus服务器不太方便，那么我们可以使用maven加载本地的jar文件

```shell
mvn install:install-file -Dfile=vertica-jdbc-9.2.0.jar -DgroupId=com.vertica -DartifactId=vertica-jdbc -Dversion=9.2.0 -Dpackaging=jar
```

### 参数说明

- -Dfile

指定本地jar文件的路径

- -DgroupId

指定本地jar的groupId

- -DartifactId

 指定本地jar的artifactId

- -Dversion

指定本地jar的version

- -Dpackaging

指定本地jar的packaging，这里使用的是`jar`，表示是一个jar文件

### pom中使用

```xml
<dependency>
  <groupId>com.vertica</groupId>
  <artifactId>vertica-jdbc</artifactId>
  <version>9.2.0</version>
</dependency>
```

