---
layout: post
title:  "简单主机内网络隔离方案"
---

# 前言

如果想隔离主机中进程的网络环境，可以使用`network namespace`（后面简称ns）来做。隔离方法很多种，本文介绍几种简单的可行方案。

# 实验环境准备

实验环境使用腾讯云上的两台cvm主机，两台机器在同一vpc同一子网下（underlay网络），这样两台机器天然内网互通。

cvm-1：

```
ubuntu@VM-6-43-ubuntu:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:9c:e7:91 brd ff:ff:ff:ff:ff:ff
    inet 10.0.6.43/24 brd 10.0.6.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe9c:e791/64 scope link
       valid_lft forever preferred_lft forever
```

cvm-2：

```
ubuntu@VM-6-46-ubuntu:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:75:e7:ca brd ff:ff:ff:ff:ff:ff
    inet 10.0.6.46/24 brd 10.0.6.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe75:e7ca/64 scope link
       valid_lft forever preferred_lft forever
```

我们之后的配置都在cvm-1上操作进行，使用cvm-2测试网络联通性。

# 方案详解

![方案分类](https://raw.githubusercontent.com/ligang1109/ligang1109.github.io/master/images/2021-03-05/fafl.png)

## 双网卡







