---
layout: post
title:  "gobox介绍"
---

从今天开始，我和大家介绍下我们自主开发的go语言轻型框架gobox，今天这一期，我先从宏观上给大家说下。

目前它的项目地址为：https://github.com/goinbox

## 前世今生

我最开始用go写的第一个项目是这边的一个代码发布系统，里面参考Raft实现了一套代码发布机故障自动检测恢复的机制。

那个时候刚刚接触go语言，开发时用到的一些库都是仿照之前用别的语言开发时的库用go重新实现的，现在放在：https://github.com/mydoraemon/golib，已经不再更新了。

随着对go越来越熟悉，项目中用的也越来越多，我们重构了这个库，并命名为gobox，放在：https://github.com/Andals/gobox，目前也已经不再更新了。

随着go官方推出了dep这个包管理工具，我们把gobox中的每一个box都单独拿出来作为一个项目管理，这就是现在的gobox：https://github.com/goinbox。

## 命名的由来

为什么叫gobox呢？因为我们设计让每一个单独的模块都作为一个box，那这些box的集合就称为gobox，再使用go的pkg管理机制引入到项目中。

## 目前支持的box

```
├── color          // 为终端输出增加颜色
├── crypto         // 常用加解密相关
├── encoding       // 常用编解码相关
├── exception      // 用errno+msg表示的错误
├── gohttp         // http服务相关
├── golog          // log记录相关
├── gomisc         // 零碎的工具库
├── go-nsq-mate    // 配合操作nsq队列
├── inotify        // 文件系统通知
├── levelcache     // 使用leveldb实现的本地cache
├── mysql          // mysql操作相关
├── page           // 分页操作
├── pool           // 连接池实现
├── redis          // redis操作相关
├── shardmap       // 为减小map锁粒度实现的碎片化map
├── shell          // 执行shell命令相关
├── simplecache    // 内存cache
```

## 使用方式

每个box都相当于一个pkg，使用也很简单，示例：

```
package main

import (
	"github.com/goinbox/color"

	"fmt"
)

func main() {
	fmt.Println(string(color.Black([]byte("Black"))))
	fmt.Println(string(color.Red([]byte("Red"))))
	fmt.Println(string(color.Green([]byte("Green"))))
	fmt.Println(string(color.Yellow([]byte("Yellow"))))
	fmt.Println(string(color.Blue([]byte("Blue"))))
	fmt.Println(string(color.Maganta([]byte("Maganta"))))
	fmt.Println(string(color.Cyan([]byte("Cyan"))))
	fmt.Println(string(color.White([]byte("White"))))
}
```

go官方提供了dep工具，配合在项目中使用可以更加方便。

dep: https://golang.github.io/dep/

## 结束语

今天就先写这些，下期开始我会分别介绍下每个box的使用方法。

欢迎大家使用，如果有遇到问题随时反馈即可，我们会快速响应！
