---
layout: post
title:  "应用编程基础课第一讲：编程基础知识"
---

本人从事linux下web编程多年，最近有幸给组内同学做培训，希望能给大家介绍下自己这些年在应用编程方面的经验，今天先给大家介绍下一些编程方面的需要掌握的基础知识:

# 操作系统介绍

先来看一个unix系统的架构图：

![arch-of-unix](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/arch-of-unix.jpg?raw=true)

从内向外，unix系统架构分为：

1. 内核：控制硬件资源，提供应用程序运行的环境
1. 系统调用：内核的编程接口
1. shell和库函数：为应用程序提供编程、运行接口
1. 应用程序：我们自己编写的程序

# 系统调用和库函数

应用程序可以调用系统调用和库函数，很多库函数都会调用系统调用，用下面这幅图来展示二者的区别：

![lib-syscall](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/lib-syscall.jpg?raw=true)

上面说过，内核用于控制硬件资源。例如从磁盘上读写文件，相当于需要控制硬盘这个硬件做IO操作，这个事情需要内核来做，那如何告诉内核要做这个事情呢？系统调用就是干这个事情的，它是内核暴露出来的一组编程接口，通过调用这个接口来执行内核中的代码。

库函数不是内核代码，是更高一层的功能封装。如我们常用的`printf`函数，我们可以调用这个函数输出内容到显示器上，但控制显示器的输出是内核做的事情，系统调用提供的是`write`方法，`printf`相当于封装了`write`这个系统调用，给应用程序提供了一个更加友好的操作方式。

总结下，系统调用相当于内核代码的调用入口，库函数是对应用程序要使用的功能的一层友好的封装。

# 用户态和内核态

程序在运行时会有用户态和内核态的区别，请见下图：

![user-kernel](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/user-kernel.jpg?raw=true)

简单一句话，程序执行时，如果执行的是我们编写的应用程序的代码，这些代码就是运行在用户态的；当代码中调用了系统调用后，接下来内核中的代码就会执行，内核中的代码就是运行在内核态的。

# 程序是如何执行的

我们用最经典的`hello world`来说明下程序是如何执行的。

程序源码`hello.c`：

```
#include <stdio.h>

int main() {
    printf("Hello, world!\n");

    return 0;
}
```

首先，我们开发的`hello.c`存储在磁盘上，我们首先编译它，得到可执行文件`hello`：

```
ligang@vm-xubuntu ~/tmp/c $ gcc -o hello hello.c 
ligang@vm-xubuntu ~/tmp/c $ ll
total 16
-rwxrwxr-x 1 ligang ligang 8296 9月  15 16:07 hello
-rw-rw-r-- 1 ligang ligang   80 9月  15 16:05 hello.c
```

这一步编译，实际上经历了如下处理流程：

![compile](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/compile.png?raw=true)

当我们运行hello程序时，效果如下：

```
ligang@vm-xubuntu ~/tmp/c $ ./hello 
Hello, world
```

实际执行流程如下：

首先，我们通过键盘输入`./hello`，shell读取每个字符到寄存器中，然后存储到主存中：

![run-hello-1](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/run-hello-1.png?raw=true)

当我们输入`enter`后，shell知道我们完成了命令输入，将hello这个程序从磁盘加载到主存：

![run-hello-2](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/run-hello-2.png?raw=true)

使用DMA（direct memory access）技术，数据从磁盘直接加载到主存中，避免了通过CPU传输。

当代码加载到主存后，CPU开始执行程序指令，这些指令拷贝`hello, world\n`字符串从内存到寄存器，然后传输到显示设备，显示设备负责显示结果：

![run-hello-3](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/run-hello-3.png?raw=true)

# 进程、线程、协程

我们先来看下一个经典的程序在内存中的布局：

![mem-arr](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/mem-arr.jpg?raw=true)

1. `text`段就是我们的程序代码
1. `initialized data`中是我们代码中明确初始化的一些全局变量、静态变量
1. `bss`段中放的是代码中没有初始化的全局变量、静态变量
1. `heap`是我们程序中动态申请的内存空间，例如调用`malloc`，这个空间很大
1. `stack`中的空间由程序中的局部变脸和函数调用使用

### 进程

什么是进程呢？我理解进程就是运行中的程序在操作系统中的一个具象化的表现。

每个进程在内存中都拥有上面那些布局，而且都是独立的，不共享。

### 线程

那么什么是线程呢？我自己理解，每个线程代表了一个独立的执行流。

举个例子，hello那个程序中，如果printf语句要执行10次，那么这10次可以一个挨着一个顺序来执行，这是一个执行流；

也可以创建10个执行流，每个都执行一次printf，如果我们有至少10个cpu核，这样就可以做到10个printf并行执行。

所以，线程是操作系统调度的最小单元。

同一进程的多线程间，栈空间（stack）是相互独立的，而其它数据是共享的，所以高性能的多线程编程首先要解决的就是锁的问题。

### 协程

一些现代的编程语言会有这个概念，它又是什么呢？

我理解，协程是一个用户态的概念。

线程是由操作系统内核进行调度的，我们无法干预，协程是用户态程序，相当于应用程序自己进行了调度。

因为它是用户态程序，所以相当于多个协程会运行在一个线程中。

要注意的是，只有内核对线程的调度才能够利用cpu的多核资源，让程序做到并行，所以在一个线程中的多个协程，是无法做到并行的。

### 父子进程间的共享

我们来看一张图： 

![open](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/open.jpg?raw=true)

这张图描述的是当我们在程序中用`open`这个系统调用打开一个文件时，操作系统中维护的相关数据结构，这里最重要的一个说明，就是最左面`process table entry`部分可以理解为是用户态程序中的，其它的`file table entry`和`v-node table entry`是内核中的。
