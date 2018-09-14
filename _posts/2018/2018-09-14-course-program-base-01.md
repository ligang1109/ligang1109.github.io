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

# 用户态和内核态

程序在运行时会有用户态和内核态的区别，请见下图：

![user-kernel](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-14/user-kernel.jpg?raw=true)
