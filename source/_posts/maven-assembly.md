---
title: maven归档插件assembly介绍
date: 2020-01-02 20:19:25
tags: maven
categories: maven
---

2020年的第一篇文章，祝大家新年快乐！



assembly插件主要用于对打包以后的文件进行归档处理，比如项目中的jar，配置文件，脚本.... 打包需要归档到一起时，就可以使用maven-assembly-plugin。assembly使用很简单，第一是需要添加对应的maven插件，第二就是添加assembly需要归档的描述文件



## 项目结构

这里用之前的一个介绍的maven多模块项目配置来进行配置

```shell
➜  life-multi-module git:(master) ✗ tree
.
├── HELP.md
├── life-multi-module.iml
├── multi-api
│   ├── multi-api.iml
│   ├── pom.xml
│   └── src
│       ├── assembly
│       │   └── assembly-descriptor.xml
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── iogogogo
│       │   │           └── MainApplication.java
│       │   ├── java-templates
│       │   │   └── com
│       │   │       └── iogogogo
│       │   │           └── Version.java
│       │   └── resources
│       │       └── git.properties
│       └── test
│           └── java
├── multi-common
│   ├── multi-common.iml
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── iogogogo
│       │   │           └── common
│       │   │               ├── CommonApplication.java
│       │   │               └── util
│       │   │                   └── IdHelper.java
│       │   └── resources
│       └── test
│           └── java
├── mvnw
├── mvnw.cmd
└── pom.xml
```



## 添加maven配置

如果是在多模块项目下，该配置放在对应模块下，不要放在parent pom.xml中，示例放在`./multi-api/pom.xml`

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-assembly-plugin</artifactId>
  <version>3.1.0</version>
  <executions>
    <execution>
      <!-- 在package任务以后执行 -->
      <phase>package</phase>
      <goals>
        <goal>single</goal>
      </goals>
      <configuration>
        <!-- 生产压缩包名称 -->
        <finalName>${project.artifactId}-${project.version}-${timestamp}</finalName>
        <appendAssemblyId>false</appendAssemblyId>
        <!-- assembly配置描述文件 -->
        <descriptors>
          <descriptor>src/assembly/assembly-descriptor.xml</descriptor>
        </descriptors>
      </configuration>
    </execution>
  </executions>
</plugin>
```



## assembly配置描述文件

```xml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2
          http://maven.apache.org/xsd/assembly-1.1.2.xsd">

    <id>release</id>
    <formats>
        <format>tar.gz</format>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>../docs/bin</directory>
            <outputDirectory>multi-api</outputDirectory>
            <fileMode>0755</fileMode>
            <includes>
                <include>*.sh</include>
            </includes>
        </fileSet>

        <fileSet>
            <directory>../docs</directory>
            <outputDirectory>multi-api</outputDirectory>
            <includes>
                <include>db/*</include>
                <include>db/*/*</include>
            </includes>
        </fileSet>

        <fileSet>
            <directory>target</directory>
            <outputDirectory>multi-api</outputDirectory>
            <includes>
                <include>*.jar</include>
                <include>lib/*</include>
            </includes>
        </fileSet>

        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>multi-api</outputDirectory>
            <includes>
                <include>logback-spring.xml</include>
                <include>application.properties</include>
            </includes>
        </fileSet>

        <fileSet>
            <directory>../</directory>
            <outputDirectory>multi-api</outputDirectory>
            <includes>
                <include>*.MD</include>
            </includes>
        </fileSet>

    </fileSets>
</assembly>
```

### 配置描述文件介绍

- id

id 标识符，添加到生成文件名称的后缀符。如果指定 id 的话，目标文件则是 ${artifactId}-${id}.tar.gz

- formats

maven-assembly-plugin 支持的打包格式有zip、tar、tar.gz (or tgz)、tar.bz2 (or tbz2)、jar、dir、war，可以同时指定多个打包格式

- includeBaseDirectory

解压时是否讲当前压缩包名称作为文件夹名称

- fileSets

管理一组文件的存放位置，核心元素如下表所示。

| **元素**          | **类型** | **作用**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| directory         | string   | 指定从哪个目录进行获取文件进行归档                           |
| outputDirectory   | string   | 指定文件集合的输出目录，该目录是相对于根目录                 |
| fileMode          | String   | 指定文件属性，使用八进制表达，分别为(User)(Group)(Other)所属属性，默认为 0644 |
| includes/include* |          | 包含文件                                                     |
| excludes/exclude* |          | 排除文件                                                     |



<br>

完整代码：https://github.com/iogogogo/life-multi-module