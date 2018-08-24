---
layout: post
title:  "gobox中的http请求处理框架"
---

今天来说下使用gobox中的http请求处理框架

## http请求处理架构图

![](https://raw.githubusercontent.com/ligang1109/ligang1109.github.io/master/images/2018-08-24-gobox-http-mvc/gobox-http-mvc.png)

## 重要的对象

#### System

system用于实现go官方包中的http.Handler接口，它的ServeHTTP方法中实现了请求处理框架。

#### Router

定义和实现MVC的路由查找过程。

```
type Router interface {
	MapRouteItems(cls ...controller.Controller)     // 自动将Controller对象中的Action方法映射到路由表
	DefineRouteItem(pattern string, cl controller.Controller, actionName string)   // 手动添加路由规则，pattern为正则表达式

	FindRoute(path string) *Route    // 实现路由查找过程
}
```

#### SimpleRouter

Router接口的一个实现，自动映射规则为：

1. controller名称规则为：`([A-Z][A-Za-z0-9_]*)Controller$`，匹配内容转小写即为controllerName
1. action名称规则为：`^([A-Z][A-Za-z0-9_]*)Action$`，匹配内容转小写后过滤掉`before和after`即为actionName


自动路由查找规则如下：

1. 将request_uri视为：`/controller/action`
1. controller不存在，则默认为`index`，可以修改
1. action不存在，则默认为`index`，可以修改

自定义路由查找规则如下：

1. 对request_uri做正则匹配
1. 如果匹配后存在捕获，则捕获内容会作为action中除context外的参数，依次传入，都是string类型

#### ActionContext和Controller

##### ActionContext

处理每个请求会创建一个对应Controller的ActionContext对象：

```
type ActionContext interface {
	Request() *http.Request
	ResponseWriter() http.ResponseWriter

	ResponseBody() []byte
	SetResponseBody(body []byte)

	BeforeAction()
	AfterAction()
	Destruct()
}
```

##### Controller

组织Action：

```
type Controller interface {
	NewActionContext(req *http.Request, respWriter http.ResponseWriter) ActionContext
}
```

#### gracehttp

这是一个支持平滑重启的httpserver，平滑重启过程如下：

![](https://github.com/ligang1109/ligang1109.github.io/blob/master/images/2018-08-24-gobox-http-mvc/gobox-gracehttp.png?raw=true)

## 示例代码

最后附上一个最简单的使用示例：

```
package main

import (
	"github.com/goinbox/gohttp/controller"
	"github.com/goinbox/gohttp/gracehttp"
	"github.com/goinbox/gohttp/router"
	"github.com/goinbox/gohttp/system"

	"net/http"
)

func main() {
	dcl := new(DemoController)
	r := router.NewSimpleRouter()

	r.DefineRouteItem("^/g/([0-9]+)$", dcl, "get")
	r.MapRouteItems(new(IndexController), dcl)

	sys := system.NewSystem(r)

	gracehttp.ListenAndServe(":8001", sys)
}

type BaseActionContext struct {
	Req        *http.Request
	RespWriter http.ResponseWriter
	RespBody   []byte
}

func (this *BaseActionContext) Request() *http.Request {
	return this.Req
}

func (this *BaseActionContext) ResponseWriter() http.ResponseWriter {
	return this.RespWriter
}

func (this *BaseActionContext) ResponseBody() []byte {
	return this.RespBody
}

func (this *BaseActionContext) SetResponseBody(body []byte) {
	this.RespBody = body
}

func (this *BaseActionContext) BeforeAction() {
	this.RespBody = append(this.RespBody, []byte(" index before ")...)
}

func (this *BaseActionContext) AfterAction() {
	this.RespBody = append(this.RespBody, []byte(" index after ")...)
}

func (this *BaseActionContext) Destruct() {
	println(" index destruct ")
}

type IndexController struct {
}

func (this *IndexController) NewActionContext(req *http.Request, respWriter http.ResponseWriter) controller.ActionContext {
	return &BaseActionContext{
		Req:        req,
		RespWriter: respWriter,
	}
}

func (this *IndexController) IndexAction(context *BaseActionContext) {
	context.RespBody = append(context.RespBody, []byte(" index action ")...)
}

func (this *IndexController) RedirectAction(context *BaseActionContext) {
	system.Redirect302("https://github.com/goinbox")
}

type DemoActionContext struct {
	*BaseActionContext
}

func (this *DemoActionContext) BeforeAction() {
	this.RespBody = append(this.RespBody, []byte(" demo before ")...)
}

func (this *DemoActionContext) AfterAction() {
	this.RespBody = append(this.RespBody, []byte(" demo after ")...)
}

func (this *DemoActionContext) Destruct() {
	println(" demo destruct ")
}

type DemoController struct {
}

func (this *DemoController) NewActionContext(req *http.Request, respWriter http.ResponseWriter) controller.ActionContext {
	return &DemoActionContext{
		&BaseActionContext{
			Req:        req,
			RespWriter: respWriter,
		},
	}
}

func (this *DemoController) DemoAction(context *DemoActionContext) {
	context.RespBody = append(context.RespBody, []byte(" demo action ")...)
}

func (this *DemoController) GetAction(context *DemoActionContext, id string) {
	context.RespBody = append(context.RespBody, []byte(" get action id = "+id)...)
}
```

运行这个代码，请求示例及输出如下：

```
curl http://127.0.0.1:8001/
index before  index action  index after 

curl http://127.0.0.1:8001/index/redirect -I
HTTP/1.1 302 Found
Content-Type: text/html; charset=utf-8
Location: https://github.com/goinbox
Date: Fri, 24 Aug 2018 11:57:11 GMT
Content-Length: 14

curl http://127.0.0.1:8001/demo/demo
demo before  demo action  demo after

curl http://127.0.0.1:8001/g/123
demo before  get action id = 123 demo after 
```

所有destruct输出为：

```
index destruct 
index destruct 
index destruct 
demo destruct 
demo destruc
```

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
