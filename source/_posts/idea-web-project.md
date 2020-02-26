---
title: IntelliJ IDEA 2019 创建maven web项目
date: 2020-02-26 22:51:03
tags:
	- Maven
	- IDEA
categories: 工具使用
---






## IntelliJ IDEA 2019 创建maven web项目

本文介绍使用IDEA创建不使用模板的web项目。



### 新建项目

![image-20200226210315550](/images/idea/image-20200226210315550.png)

![image-20200226210458711](/images/idea/image-20200226210458711.png)





### 配置项目

#### 修改项目结构设置

![image-20200226212549419](/images/idea/image-20200226212549419.png)



#### 添加web module

`Project`那边没有什么需要修改配置的地方，不过需要的话可以修改`Project compiler output`，这里我们使用默认就可以了。

此项目无任何适配服务组件（因为是手工创建Maven，没有选择任何Maven模板），因此需要我们进行添加。

这里选择一个`Web`组件就表示这是一个`web project`了



![image-20200226212710033](/images/idea/image-20200226212710033.png)



#### 配置Web Resource Directories

这里要选择`scr/main`目录，并且在后面手动添加一个`webapp`目录。

点OK，Web的资源目录便设置好了。

![image-20200226213027305](/images/idea/image-20200226213027305.png)



#### 配置Deployment Description

这一步是配置`web.xml`文件的位置，我们需要放在上一步`webapp`下面去。

![image-20200226213229541](/images/idea/image-20200226213229541.png)





#### 修改完成的结果

![image-20200226213254439](/images/idea/image-20200226213254439.png)



到这里我们可以看到底部有一个警告，是我们还没有引入`aftifact`，接下来配置`Aftifacts`。



### Aftifacts配置

这个Aftifacts描述了当前项目发布的信息。现在进行添加，从Modeles中选择。

#### 选择Modules

![image-20200226214750810](/images/idea/image-20200226214750810.png)



弹出窗直接选择我们的这个module，然后点击ok就可以了

![image-20200226214919063](/images/idea/image-20200226214919063.png)



#### 配置完成以后的结果

![image-20200226215113669](/images/idea/image-20200226215113669.png)



再回过头去看`Modules`菜单下的警告也没有了。



#### 项目结构

这里我们就可以看到我们`web`项目必须要有的`web.xml`文件，并且我们在里面添加了一个`welcome-file`，当项目启动时打开我我们的`index.html`文件。

![image-20200226215406335](/images/idea/image-20200226215406335.png)





### 配置Tomcat

#### 下载Tomcat

首先进入[Tomcat](http://tomcat.apache.org/)官网，这里我们选择了一个[Tomcat-9.0.3](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.31/bin/apache-tomcat-9.0.31.tar.gz)的版本进行下载。



#### 配置Tomcat Server

![image-20200226215940914](/images/idea/image-20200226215940914.png)

![image-20200226220025940](/images/idea/image-20200226220025940.png)





##### 配置Deployment

![image-20200226220205638](/images/idea/image-20200226220205638.png)

![image-20200226220459166](/images/idea/image-20200226220459166.png)



##### 配置Server

![image-20200226220823555](/images/idea/image-20200226220823555.png)



### 运行web项目

在webapp目录下面新建`index.html`文件，项目结构如下。

![image-20200226221200309](/images/idea/image-20200226221200309.png)

接下来启动项目

![image-20200226221359622](/images/idea/image-20200226221359622.png)

然后我们访问http://localhost:8080/，就可以看到显示了我们`index.html`的内容了。

### 新建Servlet

```java
package com.iogogogo.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * Created by tao.zeng on 2020/2/26.
 */
@WebServlet("/index")
public class IndexServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 解决乱码
        request.setCharacterEncoding(StandardCharsets.UTF_8.displayName());
        response.setCharacterEncoding(StandardCharsets.UTF_8.name());
        String name = request.getParameter("name");
        request.setAttribute("name", name);
        System.out.println(name);
        response.getWriter().println(name);
    }
}

```

然后重启项目我们在地址栏输入`http://localhost:8080/index?name=哈哈哈`就可以看到页面输出了`哈哈哈`

### 常见问题

- 无法使用`servlet`包下面的类

  解决方案：在Modules加入Tomcat依赖

![image-20200226223033946](/images/idea/image-20200226223033946.png)

![image-20200226223106321](/images/idea/image-20200226223106321.png)



![image-20200226223154510](/images/idea/image-20200226223154510.png)