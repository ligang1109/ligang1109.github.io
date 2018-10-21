---
layout: post
title:  "应用编程基础课第五讲：Go项目实践"
---

今天我给大家介绍下我使用Go语言做项目的一些心得，实际项目由于涉及公司代码，所以我这里开发了一个通用的模版项目`gobox-demo`用于说明。

gobox-demo代码：https://github.com/goinbox/gobox-demo

# 项目组织

```
├── conf           // 项目配置文件目录
│   ├── http       // httpserver（nginx）相关配置
│   │   ├── general
│   │   └── ssl
│   ├── rigger    // rigger配置
│   │   └── tpl
│   └── server
├── logs                       // 项目log文件目录
├── src                        // 里面放置go代码
│   └── gdemo                  // 相当于本项目的命名空间的概念
│       ├── conf               // 解析项目配置
│       ├── controller         // mvc中的controller
│       │   └── api            // api子系统的controller、action
│       │       ├── demo       // DemoController
│       ├── errno              // 项目的错误码
│       ├── gvalue             // 项目中用到的一些全局变量
│       ├── idgen              // id生成器，当前有个mysql的实现
│       ├── main               // 里面都是main函数，即程序执行入口
│       │   └── api            // api子系统的main
│       ├── misc               // 一些零碎的东西
│       ├── svc                // mvc中的svc层
│       │   ├── demo           // DemoSvc
│       └── vendor             // 里面防止第三方包
│           ├── github.com
│           │   ├── garyburd
│           │   ├── goinbox
│           │   └── go-sql-driver
│           └── gopkg.in
│               └── mgo.v2
├── tmp     // 项目临时文件目录 
└── tools   // 一些用到的工具脚本
```

# controller和action的组织

我见过的大多数项目，都喜欢把controlelr和action放到一个代码文件中，项目功能越多，文件就越长，

实际中这样做会给开发和维护带来很大的不利，所以我把他们拆开，每个action作为一个代码文件，这样很清晰：

```
controller/
├── api           	 // api子系统下的controller和action
│   ├── base.go   	 // api子系统的base
│   ├── demo      	 // DemoController
│   │   ├── add.go   // AddAction
│   │   ├── base.go  // DemoController的base
│   │   ├── del.go   // DelAction
│   │   ├── edit.go  // EditAction
│   │   ├── get.go   // GetAction
│   │   └── index.go // IndexAction
└── base.go   		 // 通用base
```

# svc的组织

```
svc/
├── base.go            // 通用base
├── demo               // 组织DemoSvc
│   ├── demo.go
│   └── demo_test.go
├── redis_base.go     // 提供基础redis操作
├── sql_base.go       // 提供基础sql操作
└── sql_redis_bind.go // 常用的以redis为cache + mysql存储的操作
```

# 其它

其它的一些包中都放置了线上最常用到的一些工具，我的原则向来是追求精简，团队没用到的功能就不添加。

另外，一些涉及到公司内部的代码，例如上线操作无法放到这里展示，但都是会有单独的目录去组织这些。

总之一句话，我力求做到项目组织合理，命名清晰，层次分明，希望让大家从项目的组织结构上就能判断出哪部分功能放在哪里，任何会让人有歧义的地方都要改善。

# 结束语

应用编程课已经全部讲完了，希望我的经验对大家有帮助，本人能力有限，有认识不当的地方还请指正，谢谢大家！
