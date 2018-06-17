---
layout: post
title:  "gobox中的连接池pool"
---

今天来说下gobox中的连接池底层实现pool

## 为什么需要连接池

我们的系统在访问外部资源（redis、mysql等）时，为了提高性能，通常会用到的一个优化方法就是把已经使用过的tcp连接保存起来，这样当需要再次使用时，就可以直接使用了。

这样做的好处就是减少了tcp的三次握手，也就是减少了网络间的round-trip，性能可以得到很大提升。

## 连接池配置

```
type Config struct {
	Size              int               // 最多保持多少个连接
	MaxIdleTime       time.Duration     // 每个连接的最大空闲时间，超过会自动释放
	KeepAliveInterval time.Duration     // 定义每隔多长时间进行一次空闲连接保活，这是为了防止外部资源（如redis-server会设置连接超时）主动断开连接

	NewConnFunc   func() (IConn, error)  // 从连接池中获取连接时，如果没有可用连接，会通过这个方法创建一个新连接
	KeepAliveFunc func(conn IConn) error // 连接保活时对连接池的每个连接执行此方法
}
```

## 表示每个连接的接口

```
type IConn interface {
	Free()     // 关闭连接时执行此方法
}
```

每个使用连接池的连接，实现该接口即可。

关闭连接放生在如下几种情况：

1. 连接超过最大空闲时间
2. 连接池已满时再被放入的空闲连接会被释放

## 使用方法

### import

```
import (
	"github.com/goinbox/pool"
)
```

### 新建连接池

```
func NewPool(config *Config) *Pool
```

### 从连接池中获取可用连接

```
func (p *Pool) Get() (IConn, error)
```

该方法为非阻塞方法，获取连接时若当前连接池中无可用连接，会调用config中配置的NewConnFunc方法新创建一个连接。

### 把空闲连接放回连接池

```
func (p *Pool) Put(conn IConn) error
```

该方法为非阻塞方法，放置空闲连接时若当前连接池已满，则会自动释放该空闲连接。

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
