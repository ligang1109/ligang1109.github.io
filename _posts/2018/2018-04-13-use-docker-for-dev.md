---
layout: post
title:  "使用docker高效搭建开发环境"
---

作为一个平时喜欢折腾的开发人员，我喜欢尝试各种环境，使用感兴趣的各种开源软件。

同时，我也是有一些相对的小洁癖，很喜欢linux中权限最小化原则，我也不喜欢自己的环境中有太多不知道的东西。

做了多年的web开发，我接触到的环境大致如下：

1. 操作系统从centos5到centos7；
1. webserver从apache到nginx；
1. 开发语言从最初的php5.2到php7，又到现在主要使用Go，马上还会开始接触C++；
1. 数据库从mysql5.1到现在的5.7，前阵子又开始折腾mariadb；
1. cache选型从memcache到redis；
1. 队列用过kafka，去年开始大量使用nsq；

公司虽然有专门负责部署、运维这些服务的同学，但我在开发的时候，还是喜欢自己来搭建这些东西，因为这样通常可以对使用到的服务有更多的认识，也能帮助自己使用的更好。

今天我就来和大家分享下我是如何高效的搭建好自己的开发环境的。

## 搭建前说明

这里先说明一点，对每个开源软件，我几乎都是自己编译部署的，而不会使用类似`yum install`这种方式，也很少直接下载官方编译好的二进制包，这都是为了能多深入了解用到的开源软件。

但一些依赖的动态库文件，如zlib等，还有编译工具，如gcc、make等，我都是通过方便的`yum install`这种方式直接安装的，否则会累死。

## 传统做法

我在很长的一段时间内，都是把每个软件的编译、安装过程写成一个脚本，之后再需要用的时候直接运行脚本即可，但这样的方式，通常会遇到下面这些问题：

1. 脚本只能在我当时的操作系统环境下运行。记得当时购买过不同服务商的vps，虽然不同vps我都使用同样的linux发行版，但脚本通常都不能一键跑完。这也是没办法，因为每个vps服务商都会制作自己的操作系统镜像版本。
1. 操作系统升级，如centos5 - 6，或是换为ubuntu，这样基本上脚本都跑不了。
1. 软件升级，如mysql5.2 - 5.6，构建工具改为cmake，依赖库改变或升级。
1. 如果某个软件依赖的公共库版本和其它软件不同，且公共库升级后和旧版不兼容，那你就只能为这个软件单独编译公共库了，如果只是普通的公共库还好，但如果是所需要的编译工具版本不同，那可就惨了。

上面这些问题，如果你想每个发行版维护一个脚本，那会累死，因为一旦你每次想升级一个软件，难道每个发行版都要编译一遍吗？这就变成了收获价值很低的体力劳动了。

由于喜欢折腾的个性，我对操作系统的升级以及软件包版本的升级又经常发生，所以一直以来，我都在寻找一个好方法，能很方便的维护好自己的开发环境，尽量做到每次更新东西只为它工作一次，最后我找到了docker，目前我都是用它来搭建自己的开发环境的。

## docker做法

先概括介绍下我的方法：

1. 让每个软件运行在容器中，因为运行的容器环境是可以固定下来的，所以编译安装脚本写一个就可以了。
1. 代码使用数据卷的方式加载到需要的容器中。
1. 因为是开发环境，所以网络方面使用最简单的`--net=host`。
1. 将镜像的创建、容器的启动维护在git项目中，并抽象出统一的构建过程，很方面的做到新软件接入，新机器部署。

下面用实例来说明把：

### 示例Nginx环境构建

我将构建过程放到git中：`https://gitee.com/andals/docker-nginx`

Readme中记录了构建所需要执行的脚本命令，大家访问上面的网址就可以看到，这里我简单介绍下项目的结构：

```
├── Dockerfile        //创建镜像的Dockerfile
├── pkg               //编译好的二进制包，可以直接使用，此外软件运行的一些配置文件或第三方包也放在这里
│   ├── conf
│   │   ├── fastcgi.conf
│   │   ├── http.d
│   │   ├── include
│   │   ├── koi-utf
│   │   ├── koi-win
│   │   ├── logrotate.conf
│   │   ├── logrotate.d
│   │   ├── mime.types
│   │   ├── nginx.conf
│   │   ├── scgi_params
│   │   ├── uwsgi_params
│   │   └── win-utf
│   ├── luajit-2.0.3.tar.gz
│   └── nginx-1.8.1.tar.gz
├── README.md
├── script                //里面放构建脚本
│   ├── build_image.sh    //构建镜像使用
│   ├── build_pkg.sh      //编译软件包时使用
│   ├── init.sh           //容器启动时执行
│   └── pre_build.sh      //软件依赖的共享库，编译和构建时都会用到
└── src                   //编译时需要的软件源码
    ├── modules
    │   ├── ngx_devel_kit-0.2.19.tgz
    │   ├── ngx_echo-0.53.tgz
    │   └── ngx_lua-0.9.7.tgz
    ├── nginx-1.8.1.tar.gz
    └── openssl-1.0.2h.tar.gz
```

**DockerFile说明**

Dockerfile结构如下：

```
FROM andals/centos:7
MAINTAINER ligang <ligang1109@aliyun.com>

LABEL name="Nginx Image"
LABEL vendor="Andals"

COPY pkg/ /build/pkg/
COPY script/ /build/script/

RUN /build/script/build_image.sh

CMD /build/script/init.sh
```

整个构建框架为：

1. 把构建需要的包（pkg目录中）放到镜像中
1. 把构建脚本放到镜像中
1. 执行构建脚本
1. 容器启动时，执行init.sh，里面启动相应的服务

Readme.md中记录了执行构建的命令和容器运行命令，示例运行如下：

```
ligang@vm-xubuntu16 ~/devspace/dbuild $ git clone git@gitee.com:andals/docker-nginx.git nginx
Cloning into 'nginx'...
......

ligang@vm-xubuntu16 ~/devspace/dbuild $ cd nginx/
ligang@vm-xubuntu16 ~/devspace/dbuild/nginx $ ngxVer=1.8.1
ligang@vm-xubuntu16 ~/devspace/dbuild/nginx $ docker build -t andals/nginx:${ngxVer} ./
Sending build context to Docker daemon   30.7MB
Step 1/8 : FROM andals/centos:7
......
Successfully built ea8147743031
Successfully tagged andals/nginx:1.8.1

ligang@vm-xubuntu16 ~/devspace/dbuild/nginx $ docker run -d --name=nginx-${ngxVer} --volumes-from=data-home -v /data/nginx:/data/nginx --net=host andals/nginx:${ngxVer}
dbf3c0617eb34c4b1b4ea54c2961989612d5474db3b1acd1d717221e6e5cb516
```

说明：

- `--volumes-from=data-home`这个就是我放置代码的数据卷，我喜欢把代码放到`$HOME`下面，所以这个卷的相关信息如下：

```
ligang@vm-xubuntu16 ~ $ docker ps -a
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                   PORTS               NAMES
578912a08ea7        andals/centos:7          "echo Data Volumn Ho…"   9 days ago          Exited (0) 9 days ago                        data-home
......

ligang@vm-xubuntu16 ~ $ docker inspect 578912a08ea7
......
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home",
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
......
```

- `/data/nginx`中放置nginx的conf、log等，每个软件运行时的conf、log、data等我都统一放置在/data下面，如：

```
ligang@vm-xubuntu16 ~ $ tree -d /data/ -L 1
/data/
├── mariadb
├── nginx
└── redis

ligang@vm-xubuntu16 ~ $ tree -d /data/nginx/
/data/nginx/
├── conf
│   ├── http.d
│   ├── include
│   └── logrotate.d
└── logs
```

- 启动容器时使用`--net=host`，作为开发环境简单实用

我就是通过这种方法完成了开发环境的构建，不再有多余的重复工作，并且新机器部署开发环境效率极高。

我目前用到的容器环境如下：

```
ligang@vm-xubuntu16 ~ $ docker ps -a
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                   PORTS               NAMES
dbf3c0617eb3        andals/nginx:1.8.1       "/bin/sh -c /build/s…"   33 minutes ago      Up 33 minutes                                nginx-1.8.1
3e31ef433298        andals/php:7.1.9         "/bin/sh -c /build/s…"   8 hours ago         Up 8 hours                                   php-7.1.9
360f94bf9c43        andals/jekyll:latest     "/bin/sh -c /build/s…"   9 days ago          Up 10 hours                                  jekyll-latest
0a7d58d1ca5e        andals/redis:4.0.8       "/bin/sh -c /build/s…"   9 days ago          Up 10 hours                                  redis-4.0.8
fdaa655b4a11        andals/samba:4.4.16      "/bin/sh -c /build/s…"   9 days ago          Up 10 hours                                  samba-4.4.16
6ad00a69befd        andals/mariadb:10.2.14   "/bin/sh -c /build/s…"   9 days ago          Up 10 hours                                  mariadb-10.2.14
578912a08ea7        andals/centos:7          "echo Data Volumn Ho…"   9 days ago          Exited (0) 9 days ago                        data-home
```

## 辅助工具dockerbox

使用docker环境后，有个问题，就是没有办法很方便的和软件交互。

这是因为软件都执行在容器中，比如重启nginx吧，需要下面这几步：

1. 找到nginx这个容器
1. 进入nginx这个容器
1. 在容器里面再执行reload

也可以是：

1. 找到nginx这个容器
1. 使用`docker exec`

但无论哪种方式，都比原先直接执行命令麻烦的多。

另外，有时也需要进入容器中，查看服务的运行情况。

为了方便的做这些事情，我开发了一个工具`dockerbox`，可以很方便的做到这些事情。

dockerbox的详情及使用方法请见：https://github.com/ligang1109/dockerbox

## 配置开机运行

最后再说下如何配置开机启动。

我使用虚拟机搭建的开发环境，所以配置这个会省事好多，我使用用了`systemd`：

```
ligang@vm-xubuntu16 ~ $ ll /lib/systemd/system/dstart.service
-rw-r--r-- 1 root root 229 4月   3 21:35 /lib/systemd/system/dstart.service

ligang@vm-xubuntu16 ~ $ cat /lib/systemd/system/dstart.service
[Unit]
Description=Docker Container Starter
After=network.target docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/dbox -dconfPath=/home/ligang/.dconf.json start all

[Install]
WantedBy=multi-user.target

```

dbox请参考dockerbox的使用方法。

## 结束语

上面说的是我现在使用的开发环境搭建方法，有兴趣爱折腾的同学不妨试试看，如果你有更好的方法，也希望能分享给我。

生命不息，折腾不止:-D
