---
layout: post
title:  "记一次nginx中proxy_pass的使用问题"
---

最近排查一个web服务的问题，webserver使用的nginx，最终发现是踩了nginx中proxy_pass的一个坑，这里记录下来。

# 踩坑经过

一个线上的http服务，示例nginx关键配置如下：

```
server {
    listen 80;
    server_name ligang.gdemo.com;
    server_tokens off;

    keepalive_timeout 5;

    charset utf-8;

    include /home/ligang/devspace/gobox-demo/conf/http/general/gzip.conf;

    access_log logs/ligang.gdemo.com.log combinedio buffer=1k;
    error_log  logs/ligang.gdemo.com.log.err;

    location / {
        include /home/ligang/devspace/gobox-demo/conf/http/general/http_proxy.conf;

        proxy_intercept_errors on;
        proxy_pass http://ligang.proxy.gdemo.com;
    }
}
```

这里可以看到，请求`ligang.gdemo.com`时，nginx把请求反向代理到`ligang.proxy.gdemo.com`去做处理。

`ligang.proxy.gdemo.com`这个服务在线上部署并解析到了A、B、C这3个机房，现在我想调整解析，去掉C机房，仅留A、B两个机房。

调整解析后，查看新的解析已经生效，但观察C机房的请求量，发现和之前一样，没有任何变化。

于是我观察C机房的nginx的log，发现请求来源还是`ligang.gdemo.com`的机器，域名解析调整后nginx那边依旧使用之前的IP。

于是我将`ligang.gdemo.com`的机器上的nginx全部reload后，C机房的请求终于没有了。

# 问题说明

上面的问题，说明在nginx的proxy_pass中如果使用了域名，那么nginx会把解析的结果缓存下来，貌似不会更新，因为上面的例子中，我调整解析后是几乎是隔了一天去看C机房的log发现流量没有任何变化的。

这样的话，如果你配置一个反向代理服务器，如果上游调整了域名，而你又没有得到通知，那么你的代理服务相当于不可用了。

# 从代码中看下nginx是如何解析主机ip的

有点好奇nginx是如何解析主机ip的，所以追踪下代码：

proxy_pass指令定义的地方（http/modules/ngx_http_proxy_module.c）：

```
    { ngx_string("proxy_pass"),
      NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF|NGX_HTTP_LMT_CONF|NGX_CONF_TAKE1,
      ngx_http_proxy_pass,       //处理方法
      NGX_HTTP_LOC_CONF_OFFSET,
      0,   
      NULL },
```

ngx_http_proxy_pass方法（http/modules/ngx_http_proxy_module.c）：

```
static char *
ngx_http_proxy_pass(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ngx_http_proxy_loc_conf_t *plcf = conf;

    size_t                      add;
    u_short                     port;
    ngx_str_t                  *value, *url;
    ngx_url_t                   u;
    ngx_uint_t                  n;
    ngx_http_core_loc_conf_t   *clcf;
    ngx_http_script_compile_t   sc;

	......

    url = &value[1];

	......

	ngx_memzero(&u, sizeof(ngx_url_t));

    u.url.len = url->len - add;
    u.url.data = url->data + add;
    u.default_port = port;
    u.uri_part = 1;
    u.no_resolve = 1;

	plcf->upstream.upstream = ngx_http_upstream_add(cf, &u, 0);
}

```

这里继续追踪ngx_http_upstream_add方法（http/ngx_http_upstream.c）：

```
ngx_http_upstream_srv_conf_t *
ngx_http_upstream_add(ngx_conf_t *cf, ngx_url_t *u, ngx_uint_t flags)
{
    ngx_uint_t                      i;
    ngx_http_upstream_server_t     *us;
    ngx_http_upstream_srv_conf_t   *uscf, **uscfp;
    ngx_http_upstream_main_conf_t  *umcf;

    if (!(flags & NGX_HTTP_UPSTREAM_CREATE)) {

        if (ngx_parse_url(cf->pool, u) != NGX_OK) {
            if (u->err) {
                ngx_conf_log_error(NGX_LOG_EMERG, cf, 0,
                                   "%s in upstream \"%V\"", u->err, &u->url);
            }
```

继续追踪ngx_parse_url方法（core/ngx_inet.c）：

```
ngx_int_t
ngx_parse_url(ngx_pool_t *pool, ngx_url_t *u)
{
    u_char  *p;                     
    
    p = u->url.data;
    
    if (ngx_strncasecmp(p, (u_char *) "unix:", 5) == 0) {
        return ngx_parse_unix_domain_url(pool, u);
    }
        
    if (p[0] == '[') {
        return ngx_parse_inet6_url(pool, u);
    }                              
            
    return ngx_parse_inet_url(pool, u);
}
```

然后是ngx_parse_inet_url方法（core/ngx_inet.c）：

```
static ngx_int_t
ngx_parse_inet_url(ngx_pool_t *pool, ngx_url_t *u)
{
......

    if (ngx_inet_resolve_host(pool, u) != NGX_OK) {
        return NGX_ERROR;
    }

......
}
```

然后是ngx_inet_resolve_host方法（core/ngx_inet.c）：

```
#if (NGX_HAVE_GETADDRINFO && NGX_HAVE_INET6)                                       
    
ngx_int_t                                                                          
ngx_inet_resolve_host(ngx_pool_t *pool, ngx_url_t *u)                              
{
......
    if (getaddrinfo((char *) host, NULL, &hints, &res) != 0) {
        u->err = "host not found";
        ngx_free(host);
        return NGX_ERROR;
    }
......
}

#else /* !NGX_HAVE_GETADDRINFO || !NGX_HAVE_INET6 */

ngx_int_t
ngx_inet_resolve_host(ngx_pool_t *pool, ngx_url_t *u)
{
......
        h = gethostbyname((char *) host);
......
}
```

# 思考下如何解决这个问题

最简单的解决方法，我想到如下几种：

### 执行`nginx reload`

这种方法优缺点都很明显：

- 优点

操作简单。

- 缺点

属于我们常说的后手，需要做好监控。

### 配置resolver

可以通过在nginx中配置resolver来动态更新解析，大致做法如下：

```
server {
       listen      80;
       server_name ligang.gdemo.com;

       resolver 8.8.8.8 valid=60s;
       resolver_timeout 3s;

       set $gproxy "ligang.proxy.gdemo.com";

       location / {
          proxy_pass http://$gproxy;
       }
   }
```

这个方法优缺点如下：

- 优点

解析地址每隔一段时间自动更新，无需人工做`nginx reload`。

- 缺点

需要指定DNS服务器地址，如果这个服务器挂了，或是地址变了，则需要修改nginx配置后reload。

# 结束语

上面这两个方法是无须额外开发，直接简单可用的，成本上比较低，但都有不完美的地方。

这里我想到是否可以自行开发一个nginx扩展，用来动态更新从DNS获取的IP地址，这样就能解决这个问题了，但有一定的开发成本，但个人觉得对提升技术能力又很有价值。

如果大家有什么好方法，也欢迎来一起讨论。
