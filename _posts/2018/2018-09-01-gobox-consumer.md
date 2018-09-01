---
layout: post
title:  "gobox中的consumer处理框架"
---

我们都会有从异步队列中消费的需求，今天来说下gobox中的consumer处理框架

## consumer处理架构图

![](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-09-01-gobox-consumer/gobox-consumer.png?raw=true)

## 重要的对象

#### IMessage

定义每条消息

```
type IMessage interface {
	Body() []byte
}
```

#### ConsumerHandleFunc

consumer中从队列收到每条消息后，调用这个方法

```
type ConsumerHandleFunc func(message IMessage) error
```

#### IConsumer

定义消费者行为

```
type IConsumer interface {
	SetHandleFunc(hf ConsumerHandleFunc)
	Start()
	Stop()
}
```

#### NewWorkerFunc

每个Worker的构造方法

```
type NewWorkerFunc func() IWorker
```

#### IWorker

定义Worker

```
type IWorker interface {
	SetWorkId(id int)
	SetLogger(logger golog.ILogger)

	Work(wg *sync.WaitGroup, lineCh chan []byte, stopCh chan bool)
}
```

#### LineProcessFunc

每条消息在Worker中的实际处理方法

```
type LineProcessFunc func(line []byte) error
```

#### BaseWorker

框架提供的一个简单基础Worker对象，组合这个对象后，只需要实现`LineProcessFunc`即可

```
type BaseWorker struct
```

#### Task

Task用于实现consumer的处理框架

## 使用示例

```
package main

import (
	"github.com/goinbox/goconsumer"

	"fmt"
	"strconv"
	"time"
)

// 这里实现Worker
type DemoWorker struct {
	*goconsumer.BaseWorker
}

func NewDemoWorker() goconsumer.IWorker {
	worker := &DemoWorker{goconsumer.NewBaseWorker()}
	worker.SetLineProcessFunc(worker.LineProcessFunc)

	return worker
}

func (d *DemoWorker) LineProcessFunc(line []byte) error {
	idStr := strconv.Itoa(d.Id)
	fmt.Println("wid:" + idStr + " process line:" + string(line))

	return nil
}

// 这里实现Message
type DemoMessage struct {
	body []byte
}

func (d *DemoMessage) Body() []byte {
	return d.body
}

// 这里实现一个简单的Consumer，模拟从队列中获得100条消息
type DemoConsumer struct {
	hf goconsumer.ConsumerHandleFunc
}

func (d *DemoConsumer) SetHandleFunc(hf goconsumer.ConsumerHandleFunc) {
	d.hf = hf
}

func (d *DemoConsumer) Start() {
	for i := 0; i < 100; i++ {
		str := "This message is from DemoConsumer loop " + strconv.Itoa(i)
		d.hf(&DemoMessage{[]byte(str)})
	}

	time.Sleep(time.Second * 1)
}

func (d *DemoConsumer) Stop() {

}


// 执行Task任务，调用consumer处理框架
func main() {
	task := goconsumer.NewTask("Demo")
	consumer := new(DemoConsumer)

	task.SetConsumer(consumer).
		SetWorker(10, NewDemoWorker).
		Start()
}
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
