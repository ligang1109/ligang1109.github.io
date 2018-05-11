---
layout: post
title:  "gobox中的编解码和执行shell命令"
---

今天来说下gobox中的encoding和shell两个box。

## encoding

encoding的主要作用是完成编解码工作，目前支持了base64编解码。

### 用法示例

```
package main

import (
	"github.com/goinbox/encoding"

	"fmt"
)

func main() {
	coded := encoding.Base64Encode([]byte("test goinbox base64 encoding"))
	fmt.Println("coded:", string(coded))

	uncoded := encoding.Base64Decode(coded)
	fmt.Println("uncoded:", string(uncoded))
}
```

### 输出效果示例

```
coded: dGVzdCBnb2luYm94IGJhc2U2NCBlbmNvZGluZw==
uncoded: test goinbox base64 encoding
```

## shell

shell提供了简单的执行shell命令的方法

### 简单执行命令

示例：

```
package main

import (
	"github.com/goinbox/shell"

	"fmt"
)

func main() {
	result := shell.RunCmd("ls -l /home/ligang")

	fmt.Println("exec ok:", result.Ok)
	fmt.Println("exec output:", string(result.Output))
}
```

结果输出：

```
exec ok: true
exec output: total 52
drwxrwxr-x  3 ligang ligang 4096 4月   3 17:40 backup
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:54 Desktop
drwxrwxr-x 10 ligang ligang 4096 4月  20 17:45 devspace
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Documents
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Downloads
drwxrwxr-x  4 ligang ligang 4096 4月   4 10:20 gopath
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Music
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Pictures
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Public
drwxrwxr-x  3 ligang ligang 4096 4月   3 18:12 soft
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Templates
drwxrwxr-x  9 ligang ligang 4096 5月   8 11:03 tmp
drwxr-xr-x  2 ligang ligang 4096 4月   3 17:33 Videos
```

### 以其它用户身份执行

示例：

```
package main

import (
	"github.com/goinbox/shell"

	"fmt"
)

func main() {
	result := shell.RunAsUser("ls -l /root", "root")

	fmt.Println("exec ok:", result.Ok)
	fmt.Println("exec output:", string(result.Output))
}
```

结果输出：

```
[sudo] password for ligang:
exec ok: true
exec output: total 0
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
