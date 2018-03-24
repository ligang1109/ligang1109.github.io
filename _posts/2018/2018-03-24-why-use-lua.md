---
layout: post
title:  "我们能用lua做什么"
---

lua是一个巴西人设计的小巧的脚本语言，它的设计目的是为了能够嵌入到应用程序中，从而为应用程序提供灵活的扩展和定制功能。

作为web开发工程师，我们平时主要使用的开发语言是php。这个语言提供了对html模版的强大的处理能力，也提供了十分丰富的函数库及扩展，非常的适合web开发使用。那么lua是如何进入到我们的视线中的呢？在这里我先说下我在开发一个web产品时，会优先考虑的几个问题：

1. 如何保证服务的稳定性，即如何防止白屏、50x错误的发生。 
1. 如何提高页面的响应速度，即让用户感觉页面打开足够快。 
1. 那么在用php解决这几个问题时，是否够用呢？答案在我看来是否定的，为什么这么说呢？请听我分别道来：

## php在处理服务稳定性时的不足

先简单说下由php导致的服务异常原因：

1. php语法错误、运行时的异常都会导致500错误，这在用户端浏览器就会显示出白屏。 
1. 在使用nginx + php-cgi这种组合提供动态web服务时，当后台cgi进程挂掉或数量不够用时，即会产生502错误；当php执行时遭遇阻塞（如连db时，db压力过大），而在nginx中配置的超时时间到达后，通常会产生504错误。 

这几种异常本身都是由于php导致的，当然不能靠php去解决。

webserver如nginx提供了如error_page这种用于处理当服务产生异常时的后续处理机制，为了不让用户看到白屏或错误页面，我们可以定时对正常服务时的页面做一个快照，当遇到服务异常时，就给用户这个历史快照看。

但这样做也有个局限性：当你提供服务的页面很多时，又或是需要根据请求的参数做一些逻辑上的处理时，显然就很难做到了。

你也许会说，我可以写nginx配置，让它分析请求参数，再做相应的逻辑处理。但是这样做的话，想想看你的nginx配置会有多么的复杂，多么的难以维护，而且就算你这样做，也不能解决所有的问题。比如说我有个接口要输出json字符串，或是其它别的格式，你总不能说我再去写个nginx扩展让它支持json吧。

**所以，在提高服务的稳定性方面，我们的需求是：**

1. 能用到webserver提供的错误处理机制。
1. 能方便的处理请求参数，做需要的逻辑处理。

## php在提高用户响应速度方面的不足

php是一个阻塞式顺序执行的脚本语言，虽然支持多进程执行，但这种模式并不适合使用在并发量很高的web服务中。

想像如果一个请求的处理过程中，你需要调用到多处外部资源或服务（db、rest接口），那么你的处理速度就要依赖于这些外部服务，而且是一个一个顺序处理的，它们越多，处理就越慢。

php的multi_curl可以用来并发请求这些外部的rest服务，但这样做的话，依旧需要等全部的请求都处理完成，才能返回给用户。换句话说，如果某个外部服务很慢，那么用户看到页面打开依旧会很慢。

也可以选择把页面分块，让慢的部分用js异步请求加载。但这样做的话，会增加服务器的访问量，每增加一块，访问量会增大一倍。

**所以，在提高页面的响应速度方面，我们的需求是：**

1. 耗时慢的服务能够做到异步加载，服务端每完成一部分的计算，就让页面展示这部分。 
1. 不能过大的增加服务器的压力。

## nginx-lua模块

最终，我们找到了nginx-lua?模块。这个模块会在每个nginx的worker_process中启动一个lua解释器，在nginx处理http请求的11个阶段中，你可以在其中的多个阶段用lua代码处理请求。

这二者的结合，给我们的web开发带来了新的思路。下面我就来说下导航目前是如何使用它来解决问题的。

## 解决服务稳定性

这里的思路很简单，我们会在error_page指令被执行后，用lua代码来接受参数，处理逻辑部分，最终会返回前端和用php处理看起来一致的内容。部分代码如下：

nginx_conf：

```
location ~* ^/api/.+/.+$ {
    error_page 500 502 503 504 =200 @jump_to_error_page_api;

    rewrite ^/api/(.+)/(.+)$ /index.php?_c=$1&_a=$2 break;
    root /home/ligang/demo/src/api/;

    fastcgi_pass 127.0.0.1:9000;
    include fastcgi.conf;
    fastcgi_connect_timeout 5s;
    fastcgi_send_timeout 5s;
    fastcgi_read_timeout 5s;
    fastcgi_intercept_errors on;
}

location @jump_to_error_page_api {
    lua_code_cache on;

    set $prj_home " /home/ligang/demo";

    content_by_lua_file  /home/ligang/demo/src/glue/error_page_api.lua;
}
```

这里大家看到，当请求出现50x错误时，会跳到location jump_to_error_page_api中，在这里面，content_by_lua_file指令会在content处理阶段启动指定好的lua脚本（这里是error_page_api.lua）来处理请求。我们再看下lua脚本中都做了什么：

lua示例代码：

```
ngx.header['Content-Type'] = 'text/html'
prj_home     = ngx.var.prj_home
request_args = ngx.req.get_uri_args()

local controller = request_args['_c']
local action     = request_args['_a']

if 'demo' == controller
then
    processErrorPageApiDemo(action)
else
    ngx.print('invalid controller')
end
......
```

这里大家可以看到，我们可以在lua脚本中接受请求参数，做和php一样的逻辑，最终输出前端需要的正确的内容。

目前这套机制我们已经用在我们这边的一个重要用户页面上，目前都没有收到用户反馈说页面打不开，出现错误页这种，效果很是明显。

## 提高用户页面的响应速度

上面提到的解决响应速度的几个需求，我们的思路是引入`bigpipe`的处理机制。关于bigpipe本文不做讲解，大家可以自行google。这个项目目前尚处于实验阶段，但我们已经实现了一个简单的demo:

nginx_conf：

```
location = /index.php { 
    content_by_lua_file  /home/ligang/demo/src/glue/bigpipe_index.lua;
}
```
这里指定请求index.php会用bigpipe_index.lua处理。

lua代码：

```
-- for header
ngx.header.content_type = 'text/html';
ngx.say("<html><head><title>Test Bigpipe</title></head><body>")
ngx.flush()

local capture = ngx.location.capture
local spawn = ngx.thread.spawn
local wait = ngx.thread.wait
local say = ngx.say

local function section_top()--{{{
    ngx.say("<div id='section_top'>this is top section.</div>")
    ngx.flush()
end--}}}

local function section_php(sleep) --{{{
    local res = ngx.location.capture("/content.php?sleep="..sleep)
    ngx.say(res.body)
    ngx.flush()
end--}}}

-- for body   
local threads = {
    spawn(section_top),       --这里处理没有延迟
    spawn(section_php, "1"),  --这里的处理逻辑会sleep1秒
    spawn(section_php, "5"),  --这里的处理逻辑会sleep4秒
}

for i=1, #threads do
ngx.say("<div>before wait</div>")
    wait(threads[i])
end

ngx.say("<div>all contents are loaded</div>")
ngx.say("</body></html>")
......
```

这里在处理请求时，大致逻辑如下：

1. 首先会先吐出首屏html部分及部分html框架代码。 
1. 接下来会启动3个lua协程，在nginx-lua这个模块的调度下以异步非阻塞的模式并发的来处理3个外部请求。 
1. 这3个外部请求各自的延时不同，但是任何一部分处理完成，都会直接返回给前端用于展示。 

通过这种方式，用户的页面响应速度得到了明显的提高，体验更好。

## 结束语

正如lua官方给出的定义所说，lua很小巧，非常的适合嵌入已有的应用程序中，从而补足现有系统的一些缺憾，并扩展出新的功能。对nginx-lua模块的使用，笔者也还在研究中，但我相信更好的使用它，能为我们现有的web开发打开一扇新的窗户，理解更深层次的知识。

## 参考资料

听我说完这些你有没有心动想用用看？那么在最后附上相关参考资料：

- [lua官网](https://www.lua.org/)
- [nginx-lua模块](https://github.com/openresty/lua-nginx-module)
