---
layout: post
title:  "应用编程基础课第四讲：Go编程实践"
---

今天我给大家介绍下我使用Go语言做过的一些编程实践方面。

# golog

代码在：https://github.com/goinbox/golog

无论我们做什么开发，log都是个强需求，所以先给大家介绍下我开发的golog

首先看下里面最重要的几个数据结构间的关系：

![golog](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-10-14/golog.png?raw=true)

- writer

对底层写操作的封装，写的对象可以是文件、队列、ES等，当前提供如下writer实现：

1. FileWriter写文件
1. FileWithSplitWriter写文件，可自动按照天、小时为单位分文件写入
1. ConsoleWriter写终端

- buffer

对写操作加buffer提升写性能，实现时有如下要点：

1. 实现为writer的装饰者
1. 提供单独的goroutine做autoflush

- formater

formater是将要记录的log内容发往writer之前做一次格式化，例如添加统一的log日期、终端输出添加颜色等，当前有如下实现：

1. simpleFormater在消息前面加上loglevel和时间
1. webFormater在simpleFormater的基础上添加clientIp和logId
1. consoleFormater为终端输出添加颜色

- logger

这个是程序中记录log要使用到的对象，当前提供了simpleLogger这个实现，是一种同步的方式（写操作阻塞程序执行）

- async

将写入操作放到单独的goroutine中从而提升程序性能，实现要点如下：

1. asyncLogger实现为对logger的装饰者
1. 提供单独的goroutine做写操作

更详细的使用，可以参考：https://www.jianshu.com/p/20d0f74c3c08

# shardmap

代码在：https://github.com/goinbox/shardmap

go中的原生map在多个goroutine同时读写时是需要加锁的，为了提升性能，核心思想是减少锁粒度，shardmap就是这样开发的：

![shardmap](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-10-14/shardmap.png?raw=true)

go1.8之后的官方包中提供了sync.Map用于解决map的并发读写问题，但我自己测试没有shardmap性能好，读者有兴趣可以自己试下。

更详细的使用，可以参考：https://www.jianshu.com/p/090e00f12b3e

# redis

代码在：https://github.com/goinbox/redis

redis可用的包很多，我自己实现的这个包，底层driver部分使用了redigo，考虑到实际的生产环境使用，我自行实现了如下机制：

1. 懒加载机制，即只有真正和redis做交互时才创建网络连接
1. 操作失败自动重连机制
1. 提供连接池以提高性能
1. pipeling封装
1. 事物封装

更详细的使用，可以参考：https://www.jianshu.com/p/fb498f30dff2

# goconsumer

代码在：https://github.com/goinbox/goconsumer

对异步队列的使用目前在开发中也是必不可少的，这里提供了一个消费处理框架，目前支持：

1. 消息非顺序消费
1. 消息顺序消费

整体处理框架如图：

![goconsumer](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-10-14/goconsumer.png?raw=true)

里面的对象关系如下：

- consumer

从各种队列中做消费的对象，例如kafka、nsq等

- dispatcher

分配消息到worker中处理，可以在这里实现自己的分配算法达到顺序消费的目的，当前提供下面两种实现：

1. simpleDispatcher，这个做消息的随机分发，无需顺序的消费均可以使用
1. specifyDispatcher，自己指定消息的分发方法，需要顺序消费等特殊消费需求可以使用

- worker

消息处理对象，干实际业务工作的。

- task

启动一个消费任务框架，执行的入口，我在task_test.go中有个demo实现，可供大家参考。
