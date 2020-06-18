---
title: Nginx开启gzip压缩
date: 2020-06-18 19:54:43
tags: Nginx
categories:
---

在server节点下新增如下内容，开启gzip压缩。注意后端需要保证返回的 **Content-Type: application/json;charset=UTF-8**



```shell
gzip  on;

gzip_min_length  1k;

gzip_comp_level  6;

gzip_proxied     expired no-cache no-store private auth;

gzip_types       text/plain application/x-javascript text/css application/xml application/javascript application/json;
```



![配置](/images/nginx-gzip/nginx-gzip-configure.jpg)



成功以后可以看到

**Content-Encoding: gzip**



![结果](/images/nginx-gzip/nginx-gzip-result.jpg)

