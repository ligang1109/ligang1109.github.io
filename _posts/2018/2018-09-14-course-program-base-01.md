---
layout: post
title:  "应用编程基础课第一讲：编程基础知识"
---

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
