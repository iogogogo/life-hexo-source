---
title: CentOS 升级安装最新版git
date: 2020-09-01 16:27:13
tags: Git
categories: Git
---



## 准备

安装必要依赖

```shell
yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum install -y gcc perl-ExtUtils-MakeMaker
```



## 卸载旧版本Git

如果旧版本不存在，此步可忽略

```shell
yum remove -y git
```



## 下载最新版Git

- 切换目录

```shell
cd /usr/local/src
```

- git release版本地址

```shell
https://mirrors.edge.kernel.org/pub/software/scm/git/
```

- 下载并解压

```shell
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
tar -xvf git-2.9.5.tar.gz
cd /usr/local/src/git-2.9.5
```

## 编译安装Git

```shell
make prefix=/usr/local/git all
make prefix=/usr/local/git install
```



## 配置环境变量

```shell
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile 
source /etc/profile
```

