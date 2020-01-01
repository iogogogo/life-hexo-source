---
title: mac使用parallels安装配置centos7
date: 2018-08-10 17:54:40
tags: Mac
categories: MacBook
---

### 安装parallels

https://www.jianshu.com/p/76f38e6c0792

### 配置 ssh 登录

> 1. 查看宿主机ip和网关
> 2. 修改网卡配置文件
> 3. 配置外网dns
> 4. 保存重启网络
> 5. ssh 连接 配置

### 查看宿主机IP和网关

ifconfig

### 修改网卡配置文件和NDS

> vi /etc/sysconfig/network-scripts/ifcfg-eth0

```
BOOTPROTO=static # 网卡获取IP的方式(默认为dchp,设置为静态获取。
IPADDR=192.168.1.254 # 除最后部分其他与宿主机的网关一致
GATEWAY=192.168.1.1 # 与宿主机保持一致
NETMASK=255.255.255.0
ONBOOT=yes
DNS1=192.168.1.1
DNS2=8.8.8.8
```

### 保存重启网络

```
service network restart
```

### ssh 连接 配置

```
ssh root@192.168.1.254
```

### 开放端口

- 永久的开放需要的端口

```bash
# 添加开放端口
sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent
# 关闭开放端口
sudo firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 重新加载
sudo firewall-cmd --reload
```

- 之后检查新的防火墙规则

```bash
firewall-cmd --list-all
```

### 关闭防火墙

```bash
//临时关闭防火墙,重启后会重新自动打开
systemctl restart firewalld
//检查防火墙状态
firewall-cmd --state
firewall-cmd --list-all
//Disable firewall
systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld
//Enable firewall
systemctl enable firewalld
systemctl start firewalld
systemctl status firewalld
```



