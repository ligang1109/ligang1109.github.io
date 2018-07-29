---
layout: post
title:  "gobox中redis操作"
---

今天来说下使用gobox中redis操作相关

## 说明

本包的driver部分使用了redigo：https://github.com/garyburd/redigo

### 用法示例

```
package main

import (
	"github.com/goinbox/redis"

	"time"
	"fmt"
)

func main() {
	client := redis.NewClient(redis.NewConfig("127.0.0.1", "6379", "123"), nil)

	fmt.Println("====== testGeneral ======")
	testGeneral(client)

	fmt.Println("====== testAutoReconnect ======")
	testAutoReconnect(client)

	client.Free()

}

func testGeneral(client *redis.Client) {
	reply := client.Do("set", "c", "1")
	fmt.Println(reply.String())
	reply = client.Do("get", "c")
	fmt.Println(reply.Int())

	reply = client.DoWithoutLog("set", "d", "1")
	fmt.Println(reply.String())
	reply = client.DoWithoutLog("get", "d")
	fmt.Println(reply.Int())

	client.Send("set", "a", "a")
	client.Send("set", "b", "b")
	client.Send("get", "a")
	client.Send("get", "b")
	replies, errIndexes := client.ExecPipelining()
	fmt.Println(errIndexes)
	for _, reply := range replies {
		fmt.Println(reply.String())
		fmt.Println(reply.Err)
	}

	client.BeginTrans()
	client.Send("set", "a", "1")
	client.Send("set", "b", "2")
	client.Send("get", "a")
	client.Send("get", "b")
	replies, _ = client.ExecTrans()
	for _, reply := range replies {
		fmt.Println(reply.String())
		fmt.Println(reply.Err)
	}

}

func testAutoReconnect(client *redis.Client) {
	reply := client.Do("set", "a", "1")
	fmt.Println(reply.String())
	time.Sleep(time.Second * 4) //set redis-server timeout = 3
	reply = client.Do("get", "a")
	fmt.Println(reply.Err)
	fmt.Println(reply.Int())

	time.Sleep(time.Second * 4)

	client.Send("set", "a", "a")
	client.Send("set", "b", "b")
	client.Send("get", "a")
	client.Send("get", "b")
	replies, errIndexes := client.ExecPipelining()
	fmt.Println(errIndexes)
	for _, reply := range replies {
		fmt.Println(reply.String())
		fmt.Println(reply.Err)
	}

	time.Sleep(time.Second * 4)

	client.BeginTrans()
	client.Send("set", "a", "1")
	client.Send("set", "b", "2")
	client.Send("get", "a")
	client.Send("get", "b")
	replies, _ = client.ExecTrans()
	for _, reply := range replies {
		fmt.Println(reply.String())
		fmt.Println(reply.Err)
	}

	client.Free()
}
```

程序输出：

```
====== testGeneral ======
OK <nil>
1 <nil>
OK <nil>
1 <nil>
[]
OK <nil>
<nil>
OK <nil>
<nil>
a <nil>
<nil>
b <nil>
<nil>
OK <nil>
<nil>
OK <nil>
<nil>
1 <nil>
<nil>
2 <nil>
<nil>
====== testAutoReconnect ======
OK <nil>
<nil>
1 <nil>
[]
OK <nil>
<nil>
OK <nil>
<nil>
a <nil>
<nil>
b <nil>
<nil>
OK <nil>
<nil>
OK <nil>
<nil>
1 <nil>
<nil>
2 <nil>
<nil>
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
