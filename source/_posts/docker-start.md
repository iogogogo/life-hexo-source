---
title: docke-安装
date: 2018-07-10 22:31:39
tags: docker
cover: false
categories: docker
---

### docker安装

建议在linux环境下安装Docker，window环境搭建比较复杂且容易出错，使用Centos7+yum来安装Docker环境很方便。

```shell
# 卸载旧版本docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# 安装yum-utils
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
 # 配置docker-ce.repo
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 安装docker-ce
sudo yum install docker-ce

# 启动docker
sudo systemctl start docker

# Docker守护进程自启动
sudo systemctl enable docker.service

# 显示docker版本
docker --version
```

### 设置中国镜像加速器

```shell
# 编辑daemon.json文件
vim /etc/docker/daemon.json

# 添加加速内容
{
    "registry-mirrors": ["https://registry.docker-cn.com"],
    "live-restore": true
}
```

### docker常用命令

- 拉取镜像 **docker pull [image_name]**

```shell
# 获取redis镜像
docker pull redis
Using default tag: latest
latest: Pulling from library/redis
683abbb4ea60: Pull complete
259238e792d8: Pull complete
78399601c709: Pull complete
f397da474601: Pull complete
c57de4edc390: Pull complete
b2ea05c9d9a1: Pull complete
Digest: sha256:5534b92530acc653f0721ebfa14f31bc718f68bf9070cbba25bb00bc7aacfabb
Status: Downloaded newer image for redis:latest
```

- 查看所有镜像**docker images **

```shell
# 查看所有镜像
docker images
REPOSITORY                                          TAG                 IMAGE ID            CREATED             SIZE
jenkins                                             latest              e2541428ed0d        7 days ago          696MB
redis                                               latest              71a81cb279e3        13 days ago         83.4MB
zookeeper                                           latest              397be0d8fa45        3 weeks ago         146MB
```

- 删除镜像

```
docker rmi  redis  或者  docker rmi 71a81cb279e3
```

- 查看所有运行的容器**docker ps**

```shell
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
eb5abaa4710a        mysql:5.7.20        "docker-entrypoint.s…"   6 weeks ago         Up 6 minutes        0.0.0.0:10100->3306/tcp   mysql

# 这里可以添加 -a 参数，查看所有容器，包括没有运行的
```

- 查看容器运行日志 **docker logs  -f --tail  20 [container_name] **

```shell
# -f 表示跟踪日志输出  --tail 表示持续输出 20表示第一次查看的行数 最后是容器名称
docker logs  -f --tail  20 [container_name]
2018-07-10T14:49:21.476230Z 0 [Warning] CA certificate ca.pem is self signed.
2018-07-10T14:49:21.481000Z 0 [Note] InnoDB: Buffer pool(s) load completed at 180710 
```

- 进入运行的容器**docker exec -it [container_name] bash **

当然进入容器的方式很多，但是官方推荐此方式，其他方式感兴趣可以参考此文章：  

[**nsenter进入后台运行的Docker容器**](https://github.com/cloudfstrife/note/blob/develop/docker/nsenter%E8%BF%9B%E5%85%A5%E5%90%8E%E5%8F%B0%E8%BF%90%E8%A1%8C%E7%9A%84Docker%E5%AE%B9%E5%99%A8.md)

```shell
# 进入容器
docker exec -it redis bash

# 退出容器
exit
```

- 删除所有停止的容器

```shell
docker rm $(docker ps -a -q)
```

- 删除容器

```shell
docker rm container_name/container_id
```

- 启动、停止、重启容器

```shell
docker start container_name/container_id
docker stop container_name/container_id
docker restart container_name/container_id
```

### docker安装常见问题

启动报错

```
Job for docker.service failed. See 'systemctl status docker.service' and 'journalctl -xn' for details.
```

解决方案1:

```shell
# 查看SELinux状态：
/usr/sbin/sestatus -v ##如果SELinux status参数为enabled即为开启状态
SELinux status: enabled

getenforce ##也可以用这个命令检查
# 关闭SELinux：
1、临时关闭（不用重启机器）：
setenforce 0 ##设置SELinux 成为permissive模式
##setenforce 1 设置SELinux 成为enforcing模式
2、修改配置文件需要重启机器：
修改/etc/selinux/config 文件
将SELINUX=enforcing改为SELINUX=disabled
重启机器即可
```

解决方案2:

```shell
# 升级yum，重新安装docker
yum update

# 卸载docker
sudo yum -y remove docker-ce
sudo rm -rf /var/lib/docker
# 重新安装
```

### 参考链接

[docker官网](<https://docs.docker.com/install/linux/docker-ce/centos/>)

[关闭SELinux](http://blog.51cto.com/bguncle/957315)