---
layout: post
title:  "应用编程基础课第三讲：Go编程基础"
---

上面两次课我讲解了编程方面的基础知识，这次开始，我使用Go语言来做一些编程实践方面的讲解。

今天先来说下Go语言中的一些我认为比较重要的知识点。

关于Go的基础使用，这里不做过多介绍，可以阅读：

1. How to Write Go Code：https://golang.org/doc/code.html
1. Effective Go：https://golang.org/doc/effective_go.html
1. The Way to Go：https://github.com/Unknwon/the-way-to-go_ZH_CN

# 重要的数据结构

## slice

slice是go中最常用的数据结构之一，它相当于动态数组，了解下它的内部实现，对我们是用来说有很大的好处：

slice的数据结构示例为：

```
type slice struct {
    ptr *array  //底层存储数组
    len int     //当前存储了多少个元素
    cap int     //底层数组可以存储多少个元素（从ptr指向的位置开始）
}
```

我们常用的slice有个len和cap的概念，他们就是取len和cap这两个字段的值。

slice我们通常都用它做为动态数组使用，但slice翻译过来是切片的意思，为什么呢？

我们来看个例子：

首先，我们创建一个slice：

```
s := make([]int, 5)
```

对应的数据结构为：

之后，我们再调用：

```
ss := s[2:4]
```

我们得到：

所以两个slice，相当于是在底层array上的两个切片。

# 参考

