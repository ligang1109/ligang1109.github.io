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

#### gracehttp

这是一个支持平滑重启的httpserver

欢迎大家使用，使用中有遇到问题随时反馈，我们会尽快响应，谢谢！
