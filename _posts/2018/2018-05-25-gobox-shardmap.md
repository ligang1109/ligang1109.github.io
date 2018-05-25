---
layout: post
title:  "gobox中的shardmap"
---

今天来说下gobox中的shardmap。

golang中的map使用简单，但并发写入时，如果不加锁，会导致panic，所以性能很差。

shardmap就是为了解决这个问题，其核心思想就是通过创建多个map，把key做hash分配到这多个map上，从而减少锁粒度以提高性能。

### 用法示例

```
package main

import (
	"github.com/goinbox/shardmap"
	"github.com/goinbox/crypto"

	"fmt"
	"strconv"
)

func main() {
	shardMap := shardmap.New(32)

	hasError:=false
	for i := 0; i < 10000; i++ {
		key := crypto.Md5String([]byte(strconv.Itoa(i)))
		shardMap.Set(key, i)

		v, ok := shardMap.Get(key)
		if !ok || v != i {
			hasError = true
			fmt.Println(v, ok)
		}
	}

	if !hasError {
		fmt.Println("shardmap set get ok")
	}

	shardMap.Walk(func(k string, v interface{}) {
		shardMap.Del(k)

		_, ok := shardMap.Get(k)
		if ok {
			fmt.Println(v, ok)
		}
	})
}
```

### 输出效果示例

```
shardmap set get ok
```

### 性能对比

我这里还做了和golang的原生map及sync包中提供的sync.Map的写性能对比：

- 原生map

```
BenchmarkSimpleMapWrite-4   	  300000	      5935 ns/op
```

- sync.Map

```
BenchmarkSyncMapWrite-4   	  200000	      6831 ns/op
```

- ShardMap

```
BenchmarkShardMapWrite-4   	  300000	      3815 ns/op
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
