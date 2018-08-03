---
layout: post
title:  "gobox中的分页操作"
---

今天来说下使用gobox中的分页操作

## 说明

分页也是我们开发时的一个常见需求，gobox中提供了page包做这个事情

### 用法示例

```
package main

import (
	"github.com/goinbox/page"

	"fmt"
)

func main() {
	po, err := page.NewPageObj(2, 15, 2)
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Println("current pageObj is: ", po)

	fmt.Println("iterate: ")
	po.Rewind()
	fmt.Println(po)
	for po.Next() == nil {
		fmt.Println(po)
	}
}
```

程序输出：

```
current pageObj is:  &{2 15 8 2 2}
iterate: 
&{1 15 8 2 0}
&{2 15 8 2 2}
&{3 15 8 2 4}
&{4 15 8 2 6}
&{5 15 8 2 8}
&{6 15 8 2 10}
&{7 15 8 2 12}
&{8 15 8 1 14}
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
