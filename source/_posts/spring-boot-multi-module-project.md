---
title: Maven构建Spring-Boot多模块项目
date: 2019-07-27 11:18:17
tags: Spring Boot
---

日常开发过程中肯定会用到maven构建项目，用来管理依赖的jar文件，对于使用maven构建多module项目很多时候还是不怎么熟悉，这篇文章将带着大家从头开始搭建一个使用maven构建的多module项目，并集成常用的项目插件。

## 使用Spring Initializr创建项目

这一步很简单，直接使用idea自带的功能，创建一个spring-boot项目

![image-20190715083738989](/images/multi-module/image-20190715083738989.png)



![image-20190715084023794](/images/multi-module/image-20190715084023794.png)



![image-20190715084104125](/images/multi-module/image-20190715084104125.png)



![image-20190715084159602](/images/multi-module/image-20190715084159602.png)



一个初始化的项目目录结构如下

![image-20190727103316490](/images/multi-module/image-20190727103316490.png)



## 添加新的module

一个项目中可能需要用到多个子模块，但是又有一些公共的工具类，model之类的，但是又不想传到nexus私服上面去，那么就可以添加子module，然后install以后，提供给当前project来使用了。比如我们需要一个common和一个提供api服务的module，那么可以新建两个module，parent下面的src目录也就没有存在的意义，可以直接删除。在project上面邮件新建module，步骤和新建一个maven project是一样的，这里就不做截图展示了。多module的一个完整目录结构如下：

![image-20190727103756285](/images/multi-module/image-20190727103756285.png)



## module间相互依赖

多module就是为了方便公共部分被其他module引用，其实引用也就很简单了，就类似于一个正常的maven dep就可以，比如在multi-api这个module中引用multi-common，这样一来，multi-common中的类就能被multi-api正常访问使用了

```xml
<dependency>
    <groupId>com.iogogogo</groupId>
    <artifactId>multi-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

在multi-common写了一个类叫 *com.iogogogo.common.util.IdHelper* 里面有一个获取UUID的方法，早其他已经引用multi-common的这个module中，只需要直接调用即可

```java
package com.iogogogo;

import com.iogogogo.common.util.IdHelper;
import lombok.extern.slf4j.Slf4j;

/**
 * Created by tao.zeng on 2019-07-27.
 */
@Slf4j
public class MainApplicatiom {

    public static void main(String[] args) {
        // 这里的IdHelper 类，来自于multi-common module
        log.info("uuid [{}]", IdHelper.uuid());
    }
}
```

## 打包多module项目

关于多module项目打包，spring boot升级到2.0以后，打包插件有一些小变化，比如只在multi-common加一个工具类，使用maven自带的打包工具打包，会报错误，但是不影响项目正常运行。日志如下

```verilog
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] life-multi-module
[INFO] multi-common
[INFO] multi-api
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building life-multi-module 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.6.RELEASE:repackage (repackage) @ life-multi-module ---
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building multi-common 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ multi-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ multi-common ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/tao.zeng/tmp/life-multi-module/multi-common/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ multi-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/tao.zeng/tmp/life-multi-module/multi-common/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ multi-common ---
[INFO] Changes detected - recompiling the module!
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ multi-common ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ multi-common ---
[INFO] Building jar: /Users/tao.zeng/tmp/life-multi-module/multi-common/target/multi-common-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.6.RELEASE:repackage (repackage) @ multi-common ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] life-multi-module .................................. SUCCESS [  1.037 s]
[INFO] multi-common ....................................... FAILURE [  2.536 s]
[INFO] multi-api .......................................... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.037 s
[INFO] Finished at: 2019-07-27T10:50:18+08:00
[INFO] Final Memory: 35M/293M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:2.1.6.RELEASE:repackage (repackage) on project multi-common: Execution repackage of goal org.springframework.boot:spring-boot-maven-plugin:2.1.6.RELEASE:repackage failed: Unable to find main class -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :multi-common

```

意思就是说我们在multi-common这个module中缺少了main class，那么我们加上在multi-common中添加一个类，只有一个main方法，然后再来打包，依旧报错，日志如下

```verilog
/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/bin/java -Dvisualvm.id=375540266385044 -Dmaven.multiModuleProjectDirectory=/Users/tao.zeng/tmp/life-multi-module "-Dmaven.home=/Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3" "-Dclassworlds.conf=/Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/bin/m2.conf" "-javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=56994:/Applications/IntelliJ IDEA.app/Contents/bin" -Dfile.encoding=UTF-8 -classpath "/Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/boot/plexus-classworlds-2.5.2.jar" org.codehaus.classworlds.Launcher -Didea.version=2018.3.5 -DskipTests=true package
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] life-multi-module
[INFO] multi-common
[INFO] multi-api
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building life-multi-module 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.6.RELEASE:repackage (repackage) @ life-multi-module ---
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building multi-common 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ multi-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ multi-common ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/tao.zeng/tmp/life-multi-module/multi-common/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ multi-common ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/tao.zeng/tmp/life-multi-module/multi-common/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ multi-common ---
[INFO] Changes detected - recompiling the module!
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ multi-common ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.1.2:jar (default-jar) @ multi-common ---
[INFO] Building jar: /Users/tao.zeng/tmp/life-multi-module/multi-common/target/multi-common-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.1.6.RELEASE:repackage (repackage) @ multi-common ---
[INFO] Replacing main artifact with repackaged archive
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building multi-api 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ multi-api ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ multi-api ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /Users/tao.zeng/tmp/life-multi-module/multi-api/target/classes
[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR : 
[INFO] -------------------------------------------------------------
[ERROR] /Users/tao.zeng/tmp/life-multi-module/multi-api/src/main/java/com/iogogogo/MainApplicatiom.java:[3,32] 程序包com.iogogogo.common.util不存在
[INFO] 1 error
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] life-multi-module .................................. SUCCESS [  1.499 s]
[INFO] multi-common ....................................... SUCCESS [  2.279 s]
[INFO] multi-api .......................................... FAILURE [  0.540 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.828 s
[INFO] Finished at: 2019-07-27T10:52:45+08:00
[INFO] Final Memory: 38M/302M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile) on project multi-api: Compilation failure
[ERROR] /Users/tao.zeng/tmp/life-multi-module/multi-api/src/main/java/com/iogogogo/MainApplicatiom.java:[3,32] 程序包com.iogogogo.common.util不存在
[ERROR] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :multi-api

Process finished with exit code 1

```

这个配置，在spring boot2.0版本之前，其实是完全可以的，在分析错误之前，我们先看一下整个项目的目录结构

![image-20190727105533948](/images/multi-module/image-20190727105533948.png)



可以看到在multi-common下面有两个class，在multi-api下面有一个class，那里面的内容就是引用了multi-common中的com.iogogogo.common.util.IdHelper，但是现在错误日志却说com.iogogogo.common.util不存在，这里就很纳闷了吧，接下来，介绍一下spring boot的打包插件，还得回到parent pom.xml中，使用Spring Initializr创建项目后，默认有一个 spring-boot-maven-plugin 这个maven插件，主要原因就在这里，sprint boot2.0升级以后，对插件也做了相应的升级，如果要解决这个问题，需要在此基础上做一些配置

```xml
 <build>
   <plugins>
     <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
     </plugin>
   </plugins>
</build>
```

plugin修改以后，关于configuration的配置介绍，可以看spring boot官方的文档

[maven-plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/repackage-mojo.html )


```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <configuration>
    <classifier>bootJar</classifier>
    <outputDirectory>${project.build.directory}/boot</outputDirectory>
  </configuration>
</plugin>
```

修改插件以后，在此执行打包命令就可以正常打包，我们再看一下打包以后 target目录的结果，感兴趣的同学可以去相应的目录看一下文件的大小，这里我就不在扩展了，核心日志我也有做标记

![image-20190727110524269](/images/multi-module/image-20190727110524269.png)





好了，到这里，一个完整的spring boot多module构建的项目骨架基本完成了，下面介绍一些实用的maven插件，配合spring boot食用更佳。



[github](https://github.com/iogogogo/life-multi-module)



## 常用插件使用介绍
### spring-boot-maven-plugin

该插件主要用于spring boot项目构建和打包，上面有简单的介绍和官网链接，主要就是配置

### templating-maven-plugin

该插件主要生成一个模板类，可以直接读取maven pom中的artifactId、groupId、version等信息，上面的项目源码中我会做出示例，用了该插件以后，第一次checkout project以后 需要使用maven的 compile构建一下

### buildnumber-maven-plugin

该插件主要解决maven打包以后的时间本地化问题，不然一直会有8小时的时差，一般配合maven-assembly-plugin使用更好

### maven-assembly-plugin

该插件主要用于项目打包以后的文件归档成指定的压缩包，通过配置文件自定义需要归档的文件

### maven-compiler-plugin

该插件主要规范maven project的compiler的jdk版本

### git-commit-id-plugin

该插件用于生产项目的git信息，方便程序读取

### maven-jar-plugin、maven-dependency-plugin

spring boot打包的jar默认是all in one，既一个jar包含所有依赖，可以使用这两个插件，将打出的jar做一个lib的分离