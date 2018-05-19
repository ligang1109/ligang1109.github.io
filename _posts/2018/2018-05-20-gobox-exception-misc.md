---
layout: post
title:  "gobox中的异常定义和杂项工具"
---

今天来说下gobox中的异常定义和杂项工具。

## exception

很多语言提供了异常机制，但是go没有，相似的能力可以用`panic/recover`来模拟，但是官方并不推荐这样做。

我们在系统中定义错误时通常需要错误码errno和错误信息msg，这个包就是简单的包装了下这两个常用的错误内容。

### 用法示例

```
package main

import (
	"github.com/goinbox/exception"
	
	"fmt"
)

func main() {
	e := exception.New(101, "test exception")

	fmt.Println(e.Errno(), e.Msg())
	fmt.Println(e.Error())
}
```

### 输出效果示例

```
101 test exception
errno: 101, msg: test exception
```

## gomisc

gomisc提供了很多工具方法

### slice去重

这里仅对最常用的int[]和string[]提供了方法，示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	is := []int{1, 2, 2, 3, 3, 3, 4, 4, 4, 4}
	fmt.Println("origin slice is: ", is)

	is = gomisc.IntSliceUnique(is)
	fmt.Println("after call slice is: ", is)

	ss := []string{"a", "ab", "ab", "abc", "abc", "abc", "abcd", "abcd", "abcd", "abcd", "abcd"}
	fmt.Println("origin slice is: ", ss)

	ss = gomisc.StringSliceUnique(ss)
	fmt.Println("after call slice is: ", ss)
}
```

结果输出：

```
origin slice is:  [1 2 2 3 3 3 4 4 4 4]
after call slice is:  [1 2 3 4]
origin slice is:  [a ab ab abc abc abc abcd abcd abcd abcd abcd]
after call slice is:  [a ab abc abcd]
```

### 检查文件或是目录是否存在

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	f := "/etc/passwd"

	r := gomisc.FileExist(f)
	if r {
		fmt.Println(f, "is exist")
	} else {
		fmt.Println(f, "is not exist")
	}

	d := "/home/ligang/devspace"

	r = gomisc.DirExist(d)
	if r {
		fmt.Println(d, "is exist")
	} else {
		fmt.Println(d, "is not exist")
	}
}
```

结果输出：

```
/etc/passwd is exist
/home/ligang/devspace is exist
```

### []byte追加

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	b := []byte("abc")
	b = gomisc.AppendBytes(b, []byte("def"), []byte("ghi"))

	fmt.Println(string(b))
}
```

结果输出：

```
abcdefghi
```

### 递归获取指定根目录下的所有文件，包括子目录

这里的实现，解决了目录过深时栈溢出的问题，示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	fileList, err := gomisc.ListFilesInDir("/home/ligang/tmp")
	if err != nil {
		fmt.Println(err)
		return
	}

	for _, path := range fileList {
		fmt.Println(path)
	}
}
```

输出：

```
root dir is: /home/ligang/tmp
file list under root dir are:
/home/ligang/tmp/misc/go1.10.2.linux-amd64.tar.gz
/home/ligang/tmp/misc/CLion-2018.1.2.tar.gz
/home/ligang/tmp/misc/docker-18.03.1-ce.tgz
/home/ligang/tmp/go/main.go
/home/ligang/tmp/go/.idea/modules.xml
/home/ligang/tmp/go/.idea/workspace.xml
/home/ligang/tmp/go/.idea/go.iml
```

### 保存和解析json文件

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	filePath := "/tmp/test_save_parse_json_file.json"

	v1 := make(map[string]string)
	v1["k1"] = "a"
	v1["k2"] = "b"
	v1["k3"] = "c"

	err := gomisc.SaveJsonFile(filePath, v1)
	if err != nil {
		fmt.Println("save json file failed: " + err.Error())
	} else {
		fmt.Println("save json file success")
	}

	v2 := make(map[string]string)
	err = gomisc.ParseJsonFile(filePath, &v2)
	if err != nil {
		fmt.Println("parse json file failed: " + err.Error())
	} else {
		fmt.Println("parse json file success")
	}

	for k, v := range v2 {
		if v != v1[k] {
			fmt.Println("save parse json file error, k: " + k + " not equal")
		} else {
			fmt.Println("save parse json file k:", k, "equal")
		}
	}
}
```

输出：

```
save json file success
parse json file success
save parse json file k: k2 equal
save parse json file k: k3 equal
save parse json file k: k1 equal
```

### 字符串截取

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	s := "abcdefg"

	_, err := gomisc.SubString(s, 3, 20)
	fmt.Println(err)

	_, err = gomisc.SubString(s, 10, 3)
	fmt.Println(err)

	ss, _ := gomisc.SubString(s, 3, 4)
	fmt.Println(s, "substr", "3,4", "is", ss)
}
```

输出：

```
end too long
end too long
abcdefg substr 3,4 is defg
```

### 时间格式化时的常量定义

```
TIME_FMT_STR_YEAR   = "2006"
TIME_FMT_STR_MONTH  = "01"
TIME_FMT_STR_DAY    = "02"
TIME_FMT_STR_HOUR   = "15"
TIME_FMT_STR_MINUTE = "04"
TIME_FMT_STR_SECOND = "05"
```

### 常用时间格式化时的格式

输出格式：`yyyy-mm-dd h:i:s`

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
	"time"
)

func main() {
	layout := gomisc.TimeGeneralLayout()
	fmt.Println("fmt layout is", layout)

	fmt.Println("not time is", time.Now().Format(layout))
}
```

输出：

```
fmt layout is 2006-01-02 15:04:05
not time is 2018-05-20 06:15:08
```

### 通过时间生成随机数

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
	"time"
)

func main() {
	tm := time.Now()
	fmt.Println(gomisc.RandByTime(&tm), gomisc.RandByTime(&tm), gomisc.RandByTime(nil))
}
```

输出：

```
4423491624236117727 4423491624236117727 1471010178475409526
```

这里请注意：时间值相同时运算结果是相同的。

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
