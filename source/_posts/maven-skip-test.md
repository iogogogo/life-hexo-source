---
title: maven.test.skip和skipTests的区别
date: 2020-08-20 22:15:07
tags: Maven
categories: 
	- Maven
---

## -DskipTests

不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下。

- `pom`中配置跳过

```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skipTests>true</skipTests>  
    </configuration>  
</plugin> 
```





## -Dmaven.test.skip=true

不执行测试用例，也不编译测试用例类，不但跳过单元测试的运行，也跳过测试代码的编译。

- `pom`中配置跳过

```xml
<plugin>  
    <groupId>org.apache.maven.plugin</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>2.1</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin>  
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <version>2.5</version>  
    <configuration>  
        <skip>true</skip>  
    </configuration>  
</plugin> 
```

