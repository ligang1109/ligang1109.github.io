---
layout: post
title:  "如何开发一个TcpServer -- 第二篇"
---

上一次，我们开发了一个最简单的服务端程序。

这个程序有个问题，当服务端在处理一个连接的时候，其它的连接无法得到同时处理，也就是我们常说的，这不是个高并发服务程序。

本期我们来开始解决这个问题。

# 服务端并发模型

服务端编程发展到今天，并发模型这块，也是很成熟了。

cppbox使用的是单进程多线程的模式，每个线程都有各自单独的事件循环，这也是无锁化编程的基础。

![cppbox多线程](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2019-06-29/multi_thread.png?raw=true)

## ListenThread

这个线程在做的事情如下：

1. 调用`accept`获得`connfd`
1. 将这个`connfd`分配给一个`ConnectionThread`

## ConnectionThread

# 事件循环

开源的事件循环库
