---
layout: post
title:  "gobox中的simplecache和levelcache"
---

今天来说下gobox中的simplecache和levelcache

## simplecache

simplecache提供了一个简单的内存kv

### 用法示例

```
package main

import (
	"github.com/goinbox/crypto"
	"github.com/goinbox/simplecache"

	"fmt"
	"time"
	"strconv"
)

func main() {
	sc := simplecache.NewSimpleCache()

	for i := 0; i < 10000; i++ {
		key := crypto.Md5String([]byte(strconv.Itoa(i)))
		sc.Set(key, i, 3*time.Second)

		v, ok := sc.Get(key)
		if !ok || v != i {
			fmt.Println(v, ok)
		}
	}

	time.Sleep(4 * time.Second)

	allKeysExpires := true
	for i := 0; i < 10000; i++ {
		key := crypto.Md5String([]byte(strconv.Itoa(i)))

		v, ok := sc.Get(key)
		if ok || v == i {
			fmt.Println(v, ok)
			allKeysExpires = false
		}
	}

	if allKeysExpires {
		fmt.Println("all keys have expired")
	}
}
```

### 输出效果示例

```
all keys have expired
```

### 特别说明

1. 在使用时，如果set的值是引用类型，那么改变引用的对象的值时，cache中的内容也会改变，这一点要特别注意。
2. 因为是内存cache，所以进程退出则cache内容全部丢失。

## levelcache

levelcache以leveldb为底层存储引擎，提供了一个单机落盘的kv

### 用法示例

```
package main

import (
	"github.com/goinbox/levelcache"

	"fmt"
	"time"
)

func main() {
	cache, _ := levelcache.NewCache("/tmp/levelcache_test", 5*time.Second)

	key := []byte("k1")
	value := []byte("v1")

	cache.Set(key, value, 3)

	value, _ = cache.Get(key)
	sv := string(value)
	fmt.Println(sv)
	if sv != "v1" {
		fmt.Println("set get error")
	}

	time.Sleep(4 * time.Second)

	v, err := cache.Get(key)
	sv = string(v)
	fmt.Println(sv, err)
}
```

### 输出效果示例

```
v1
 <nil>
```

### 特别说明

cache会存储在单机磁盘上，不是分布式的。

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
