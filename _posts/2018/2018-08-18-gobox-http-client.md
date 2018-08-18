---
layout: post
title:  "gobox中的httpclient"
---

今天来说下使用gobox中httpclient，这个包就相当于命令行的curl工具，用于发起http请求。

## 重要的对象

#### config

```
const (
	DEFAULT_TIMEOUT        = 30 * time.Second
	DEFAULT_KEEPALIVE_TIME = 30 * time.Second

	DEFAULT_MAX_IDLE_CONNS_PER_HOST = 10
)

type Config struct {
	LogLevel int

	Timeout       time.Duration      // 连接及读写超时
	KeepAliveTime time.Duration

	MaxIdleConnsPerHost int
}
```

#### request

```
type Request struct {
	Method     string
	Url        string
	Body       []byte
	Ip         string     // 相当于设置hostIp
	ExtHeaders map[string]string

	*http.Request
}
```

#### response

```
type Response struct {
	T        time.Duration  // 请求耗时
	Contents []byte         // 响应内容

	*http.Response
}
```

## 代码示例

```
package main

import (
	"github.com/goinbox/gohttp/httpclient"

	"time"
	"net/http"
	"fmt"
)

func main() {
	config := httpclient.NewConfig()
	config.Timeout = time.Second * 1

	client := httpclient.NewClient(config, nil)

	fmt.Println("clientGet")
	clientGet(client)

	fmt.Println("clientPost")
	clientPost(client)
}

func clientGet(client *httpclient.Client) {
	extHeaders := map[string]string{
		"GO-CLIENT-1": "gobox-httpclient-1",
		"GO-CLIENT-2": "gobox-httpclient-2",
	}
	req, _ := httpclient.NewRequest(http.MethodGet, "http://www.vmubt.com/test.php?a=1&b=2", nil, "127.0.0.1", extHeaders)

	resp, err := client.Do(req, 1)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(string(resp.Contents), resp.T.String())
	}
}

func clientPost(client *httpclient.Client) {
	extHeaders := map[string]string{
		"GO-CLIENT-1":  "gobox-httpclient-1",
		"GO-CLIENT-2":  "gobox-httpclient-2",
		"Content-Type": "application/x-www-form-urlencoded;charset=utf-8",
	}
	params := map[string]interface{}{
		"a": 1,
		"b": "bb",
		"c": "测试post",
	}
	req, _ := httpclient.NewRequest(http.MethodPost, "http://www.vmubt.com/test.php", httpclient.MakeRequestBodyUrlEncoded(params), "127.0.0.1", extHeaders)

	resp, err := client.Do(req, 1)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println(string(resp.Contents), resp.T.String())
	}
}
```

输出：

```
clientGet
array(0) {
}
 1.516315ms

clientPost
array(3) {
  ["a"]=>
  string(1) "1"
  ["b"]=>
  string(2) "bb"
  ["c"]=>
  string(10) "测试post"
}
 1.616384ms
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
