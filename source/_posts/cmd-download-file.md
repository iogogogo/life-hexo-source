---
title: 使用wget或者curl下载github release文件
date: 2020-02-14 12:22:38
tags: linux
categories: tools
---



有时候需要在服务器下载GitHub上的release资源，这时候我们可以使用`wget`或者`curl`进行处理，这里拿携程开源的配置中心[Apollo](https://github.com/ctripcorp/apollo)为例，下载他的release版本



## wget

`wget --no-check-certificate --content-disposition https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-adminservice-1.5.1-github.zip`

## curl

`curl -LJO https://github.com/ctripcorp/apollo/releases/download/v1.5.1/apollo-adminservice-1.5.1-github.zip`



