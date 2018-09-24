---
layout: post
title:  "应用编程基础课第二讲：网络编程基础"
---

今天给大家介绍下一些网络编程方面的需要掌握的基础知识:

# 网络分层模型

先来看一张图：

![osi-tcp-ip](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/osi-tcp-ip.png?raw=true)

从左到右向，分别是：

1. OSI七层模型
1. TCP/IP四层模型
1. 应用程序实现部分和内核实现部分

这里要认识到的是，我们最常用TCP的网络处理部分，都是由内核来完成的。

# TCP服务端和客户端编程模型

## TCP连接创建和断开

TCP创建连接需要三次握手，而断开连接需要四次挥手，如图：

![tcp-3-4](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/tcp-3-4.png?raw=true)

这张图清晰的说明了连接的建立、数据发送以及断开连接时所对应的编程函数，另外还有相应的TCP状态转换。

## 服务端客户端编程函数

![client-server](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/client-server.png?raw=true)

由此可见，服务端编程用到的主要函数为：

1. socket：创建一个socket，返回的文件描述符fd之后用于bind和listen
1. bind：绑定socket和ip+port
1. listen：调用后，服务端状态变为`LISTEN`，可以接收网络连接
1. accept：函数在连接建立后返回一个connfd，对这个文件描述符的读写就是在做网络接收和发送
1. read：网络对端发送来的数据会放到内核的接收缓冲区，read就是从这个缓冲区中读取数据到应用程序
1. write：应用程序要发送数据到网络对端时，调用此函数，会现将数据写到内核的发送缓冲区中，之后内核会负责将数据发送给网络对端
1. close：关闭连接

关于服务端的用于连接的fd和用于读写的fd，请见下图：

![listenfd](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/listenfd.png?raw=true)

![connfd](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/connfd.png?raw=true)

客户端编程用到的函数为：

1. socket：创建一个socket，之后用于连接服务器，做数据读写用
1. connect：发起到服务端的链接，返回时TCP三次握手完成
1. write：同服务端的write
1. read：同服务端的read
1. close：关闭连接

# 几个概念

## backlog

先附上一个图：

![backlog](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/backlog.png?raw=true)

内核中会维护两个队列：

1. 未完成连接的队列： 服务端收到客户端的连接请求（SYNC），在三次握手完成前，会放到这个队列中
1. 完成连接的队列：完成三次握手后就创建了一个TCP连接，这个连接会放到这个队列中

这两个队列中的连接数总和，就是backlog，也就是说，backlog定义了服务端可以同时维护的最大TCP连接数量

## RTT

这个概念常常听到，请见图：

![rtt](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/rtt.png?raw=true)

RTT的定义是`Round-Trip Time`，即数据包的往返时延

## TCP Stream

我们通常说TCP是流式的，这是什么样的概念呢？

![tcp-stream](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/socket-stream.png?raw=true)

对应一个TCP连接，内核会给这个连接分配一个发送缓冲和接收缓冲，我们的应用程序对这来两个缓冲区的读写就是在做网络数据的接收和发送。

而数据在网络上的传输是内核自己在维护的，当发送缓冲区中有数据后，内核就把这些数据发送给对端；同样的，当对端有数据过来时，内核会把它放到接收缓冲区中，等待应用程序的读写。

这样的发送和接收数据的过程，就像水流一样，所以我们说TCP是流式的。

## TCP状态转换

TCP定义了很多状态，这些状态之间的转换关系如下图：

![tcp-status](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/tcp-status.png?raw=true)

这些状态都记住有难度，需要时查下就好了。

# IO模型

网络操作就是IO操作，而且网络的IO是最慢的一种了，网络编程的很大难点就是妥善的处理好这一块的问题。

Unix定义了多种IO操作模型，分别是：

1. 阻塞IO
1. 非阻塞IO
1. IO多路复用
1. 信号驱动IO
1. 异步IO

分别说明如下：

## 阻塞IO

这里有一点非常重要的概念要先说明下，那就是`阻塞`的是什么？

![io-block](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/io-block.png?raw=true)

首先要记住：数据在网络上的传输完全是内核在控制的，应用程序中的read和write只是在读写接收缓冲和发送缓冲。

读阻塞：调用read时，如果接收缓冲区中没有数据，那么就会产生阻塞

写阻塞：调用write时，如果发送缓冲区中的数据没有发送出去，那么就会产生阻塞

那么如果没有产生阻塞，那么两者的执行时间为：

read的执行时间为：将内核接收缓冲区的内容拷贝到用户空间中的应用程序缓冲区

write的执行时间为：将用户空间中的应用程序缓冲区的内容拷贝到内核的发送缓冲区

## 非阻塞IO

理解了导致阻塞的原因，那么非阻塞就非常好理解了：

![io-nonblock](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/io-nonblock.png?raw=true)

当内核缓冲区无法读写时，read和write就会返回`EWOULDBLOCK`，应用程序就需要过会儿再来操作。

光是这样，还不能实现高性能的网络程序，这是因为我们无法判断应该什么时间再来做读写操作。

如果一直循环读写，那么CPU占用会很居高不下；如果sleep一段时间，那么多长时间合适呢？

所以，如果想开发高性能的网络程序，我们还需要别的武器：

## IO多路复用

这是操作系统提供的一种通知机制，告诉应用程序何时可以做读写操作：

![io-multiple](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/io-multiple.png?raw=true)

不同操作系统提供了不同的编程接口，一个非常有名的库`libevent`就是对这些库的一个统一接口封装。

IO多路复用也是现在用的最多的一种高性能网络服务器的IO处理模型，例如`Nginx`

## 信号驱动IO

不同于IO多路复用，操作系统用信号的方式告诉应用程序何时可以做读写操作：

![io-signal](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/io-signal.png?raw=true)

## 异步IO

最后这一种我没有用过，从概念上理解，相当于操作系统将数据做完用户空间和内核空间的复制后，才会通知应用程序：

![io-async](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-23/io-async.png?raw=true)

# 结束语

上面这些，都是笔者编程这些年，觉得非常受用的基础知识。正确的认识这些知识，很多问题你都可以自己想明白了。

笔者也是在不断学习中，如果有错误的地方，还望指正，我们共同进步，谢谢！

# 参考

UNIX网络编程（卷1）：https://book.douban.com/subject/4859464/

TCP/IP详解（卷1）：https://book.douban.com/subject/26790659/

Linux/UNIX系统编程手册：https://book.douban.com/subject/25809330/
