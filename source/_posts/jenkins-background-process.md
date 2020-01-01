---
title: Jenkins 后台进程
date: 2019-02-13 09:58:53
tags: Jenkins
categories: Jenkins
---

配置 Jenkins Job 的时候，after `mvn package` 用命令行 `nohup java -jar project-1.0-SNAPSHOT.jar > /dev/null 2>&1 & `起一个 spring-boot 项目，死活不生效。

ssh 到服务器上查不到对应的进程，而直接在服务器上执行是完全 OK 的。

Google 一番后得知，这是 Jenkins 的特性。

> To reliably kill processes spawned by a job during a build, Jenkins contains a bit of native code to list up such processes and kill them. This is tested on several platforms and architectures, but if you find a show-stopper problem because of this, you can disable this feature by setting a Java property named “hudson.util.ProcessTree.disable” to the value “true”. This can be done as a parameter to the “java” binary when starting Jenkins:

```
`java -Dhudson.util.ProcessTree.disable=true -jar jenkins.war`
```

> The ProcessTreeKiller takes advantage of the fact that by default a new process gets a copy of the environment variables of its spawning/creating process.

> It sets a specific environment variable in the process executing the build job. Later, when the user requests to stop the build job’s process it gets a list of all processes running on the computer and their environment variables, and looks for the environment variable that it initially set for the build job’s process.

具体链接请点击 [ProcessTreeKiller](https://wiki.jenkins-ci.org/display/JENKINS/ProcessTreeKiller)。

大概意思是 Jenkins 是在启动 Job 的时候会给子进程设置环境变量，在结束 Job 的时候会检查进程的环境变量，如果包含 Jenkins 生成的，kill 掉。

解决方案：

1. 启动 Jenkins 的时候加上 `-Dhudson.util.ProcessTree.disable=true`，也就是 `java -Dhudson.util.ProcessTree.disable=true -jar jenkins.war`
2. 在后台进程前加上 `BUILD_ID=dontKillMe`, 也就是 `BUILD_ID=dontKillMe nohup java -jar project-1.0-SNAPSHOT.jar > /dev/null 2>&1 &`

除了上面两种方式，网上也有说讲nohup换成 setsid的方式，如有需要，可自行测试验证