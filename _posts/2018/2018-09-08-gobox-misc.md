---
layout: post
title:  "gobox中的常用工具包gomisc"
---

有一些常用的工具函数，我们把它们放到gomisc这个包中。

## 工具介绍

#### Slice中的值Unique

```
func IntSliceUnique(s []int) []int
func StringSliceUnique(s []string) []string
```

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	TestIntSliceUnique()
	TestStringSliceUnique()
}

func TestIntSliceUnique() {
	s := []int{1, 2, 2, 3, 3, 3, 4, 4, 4, 4}

	fmt.Println("origin slice is: ", s)

	s = gomisc.IntSliceUnique(s)

	fmt.Println("after call slice is: ", s)
}

func TestStringSliceUnique() {
	s := []string{"a", "ab", "ab", "abc", "abc", "abc", "abcd", "abcd", "abcd", "abcd", "abcd"}

	fmt.Println("origin slice is: ", s)

	s = gomisc.StringSliceUnique(s)

	fmt.Println("after call slice is: ", s)
}
```

输出：

```
origin slice is:  [1 2 2 3 3 3 4 4 4 4]
after call slice is:  [1 2 3 4]
origin slice is:  [a ab ab abc abc abc abcd abcd abcd abcd abcd]
after call slice is:  [a ab abc abcd]
```

#### 文件、目录是否存在

```
func FileExist(path string) bool
func DirExist(path string) bool
```

示例：

```
package main

import (
	"github.com/goinbox/gomisc"

	"fmt"
)

func main() {
	TestFileExist()
	TestDirExist()
}

func TestFileExist() {
	f := "/etc/passwd"

	r := gomisc.FileExist(f)
	if r {
		fmt.Println(f, "is exist")
	} else {
		fmt.Println(f, "is not exist")
	}
}

func TestDirExist() {
	d := "/home/ligang/devspace"

	r := gomisc.DirExist(d)
	if r {
		fmt.Println(d, "is exist")
	} else {
		fmt.Println(d, "is not exist")
	}
}
```

输出：

```
/etc/passwd is exist
/home/ligang/devspace is exist
```

#### 多个[]byte拼接

```
func AppendBytes(b []byte, elems ...[]byte) []byte
```

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

输出：

```
abcdefghi
```

#### 列出目录中的所有文件（包括子目录）

```
func ListFilesInDir(rootDir string) ([]string, error)
```

示例：

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
/home/ligang/tmp/misc/accounts.json
/home/ligang/tmp/go/main.go
/home/ligang/tmp/go/.idea/modules.xml
/home/ligang/tmp/go/.idea/workspace.xml
/home/ligang/tmp/go/.idea/go.iml
```

#### 保存/解析json文件

```
func SaveJsonFile(filePath string, v interface{}) error
func ParseJsonFile(filePath string, v interface{}) error
```

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
	}

	v2 := make(map[string]string)
	err = gomisc.ParseJsonFile(filePath, &v2)
	if err != nil {
		fmt.Println("parse json file failed: " + err.Error())
	}

	ok := true
	for k, v := range v2 {
		if v != v1[k] {
			ok = false
			fmt.Println("save parse json file error, k: " + k + " not equal")
		}
	}
	if ok {
		fmt.Println("save parse json file ok")
	}
}
```

输出：

```
save parse json file ok
```

#### 字符串截取

```
func SubString(str string, start, cnt int) (string, error)
```

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
	fmt.Println(ss)
}
```

输出：

```
end too long
end too long
defg
```

#### 时间格式化定义

```
const (
	TIME_FMT_STR_YEAR   = "2006"
	TIME_FMT_STR_MONTH  = "01"
	TIME_FMT_STR_DAY    = "02"
	TIME_FMT_STR_HOUR   = "15"
	TIME_FMT_STR_MINUTE = "04"
	TIME_FMT_STR_SECOND = "05"
)

// 时间格式：yy-mm-dd H:i:s
func TimeGeneralLayout() string {
	layout := TIME_FMT_STR_YEAR + "-" + TIME_FMT_STR_MONTH + "-" + TIME_FMT_STR_DAY + " "
	layout += TIME_FMT_STR_HOUR + ":" + TIME_FMT_STR_MINUTE + ":" + TIME_FMT_STR_SECOND

	return layout
}
```

#### 生成随机数

```
func RandByTime(t *time.Time) int64
```

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
6537266505970973466 6537266505970973466 5616191991172093652
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
