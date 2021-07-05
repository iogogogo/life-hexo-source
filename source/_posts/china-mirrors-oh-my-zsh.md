---
title: oh-my-zsh国内镜像安装和更新方法
date: 2021-07-05 23:43:13
tags: zsh
categories: zsh
---

## 安装依赖

1. git
2. zsh



## 安装git

[CentOS 升级安装最新版git](https://iogogogo.gitee.io/2020/09/01/soft-install-git/)



## 下载码云安装包

```sh
wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh
```



## 编辑install.sh

找到以下部分

```sh
# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
```

把

```sh
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
```

替换为

```sh
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```

编辑后保存, 运行安装即可. (运行前先给install.sh权限)



## 修改仓库地址

```bash
cd ~/.oh-my-zsh
git remote set-url origin https://gitee.com/mirrors/oh-my-zsh.git
git pull
```



## oh-my-zsh升级

```shell
omz update # 推荐使用

# 或者

upgrade_oh_my_zsh # 官方不推荐使用
```



原文地址: [https://touka.dev/tech/oh-my-zsh-china-mirror/](https://links.jianshu.com/go?to=https%3A%2F%2Ftouka.dev%2Ftech%2Foh-my-zsh-china-mirror%2F)



