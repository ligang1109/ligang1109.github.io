---
layout: post
title:  "gobox中处理文件系统通知"
---

今天来说下使用gobox中inotify来处理文件系统通知

## inotify介绍

inotify是linux系统的文件事件通知机制，例如我们想得知某个文件是否被改变，目录的读写情况等，都可以使用这个机制。

### 用法示例

```
package main

import (
	"github.com/goinbox/inotify"

	"fmt"
	"path/filepath"
)

func main() {
	path := "/tmp/a.log"

	watcher, _ := inotify.NewWatcher()
	watcher.AddWatch(path, inotify.IN_ALL_EVENTS)
	watcher.AddWatch(filepath.Dir(path), inotify.IN_ALL_EVENTS)

	for i := 0; i < 5; i++ {
		events, _ := watcher.ReadEvents()
		for _, event := range events {
			if watcher.IsUnreadEvent(event) {
				fmt.Println("it is a last remaining event")
			}
			showEvent(event)
		}
	}

	watcher.Free()
	fmt.Println("bye")
}

func showEvent(event *inotify.Event) {
	fmt.Println(event)

	if event.InIgnored() {
		fmt.Println("inotify.IN_IGNORED")
	}

	if event.InAttrib() {
		fmt.Println("inotify.IN_ATTRIB")
	}

	if event.InModify() {
		fmt.Println("inotify.IN_MODIFY")
	}

	if event.InMoveSelf() {
		fmt.Println("inotify.IN_MOVE_SELF")
	}

	if event.InMovedFrom() {
		fmt.Println("inotify.IN_MOVED_FROM")
	}

	if event.InMovedTo() {
		fmt.Println("inotify.IN_MOVED_TO")
	}

	if event.InDeleteSelf() {
		fmt.Println("inotify.IN_DELETE_SELF")
	}

	if event.InDelete() {
		fmt.Println("inotify.IN_DELETE")
	}

	if event.InCreate() {
		fmt.Println("inotify.IN_CREATE")
	}
}
```

如示例，我们监听了`/tmp/a.log`和`/tmp`目录的文件系统事件。

执行程序，接下来我们做如下操作：

```
ligang@vm-xubuntu /tmp $ echo a > a.log
ligang@vm-xubuntu /tmp $ echo aa >> a.log
ligang@vm-xubuntu /tmp $ mv a.log b.log
ligang@vm-xubuntu /tmp $ rm b.log
rm: remove regular file 'b.log'? y
```

得到输出：

```
&{1 256 0 /tmp a.log}
inotify.IN_CREATE
&{1 32 0 /tmp a.log}
&{1 2 0 /tmp a.log}
inotify.IN_MODIFY
&{1 8 0 /tmp a.log}
&{1 32 0 /tmp a.log}
&{1 2 0 /tmp a.log}
inotify.IN_MODIFY
&{1 8 0 /tmp a.log}
&{1 64 1801 /tmp a.log}
inotify.IN_MOVED_FROM
&{1 128 1801 /tmp b.log}
inotify.IN_MOVED_TO
&{1 512 0 /tmp b.log}
inotify.IN_DELETE
bye
```

inotify对系统编程开发有着很大的意义， 如果对inotify不是很熟悉，我在这里推荐阅读`Linux/UNIX系统编程手册：https://book.douban.com/subject/25809330/`

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
