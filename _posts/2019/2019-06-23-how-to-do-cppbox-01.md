---
layout: post
title:  "如何开发一个TcpServer -- 第一篇"
---

今年开始我这边在用C++写一个TcpServer，包括核心的网络框架部分，目前开发接近完成，所以从这篇文章开始和大家分享一下我是如何从0到1完成这个TcpServer的。

这是我开发的网络框架cppbox：https://github.com/ligang1109/cppbox，期待交流

# 为什么做这个

可能有人想问，现在有很多开源的网络框架啊，为啥你还要自己做一个呢？这不是重复造轮子吗？

这里我说下自己这样做的原因：

1. 这类需求很多，但团队内部并没有一个统一的框架传承下来，在不断开发维护
1. 编程细节很重要，不了解核心细节是cover不了一个项目的。由于之前自己并没有这方面的开发经验，直接用开源的话由于不能理解核心细节，所以出问题的话会很难定位，而且对潜在的风险也一无所知
1. 一个精简的网络框架的开发难度并不算很高，毕竟不是要从头开发一个Nginx，所以还是可以接受的
1. 对团队的技术积累是有好处的

# 预备知识

先说下需要具备的基础知识，之前我写过两篇文章：

1. 应用编程基础课第一讲：编程基础知识：http://blog.7rule.com/2018/09/14/course-program-base-01.html
1. 应用编程基础课第二讲：网络编程基础：http://blog.7rule.com/2018/09/23/course-program-base-02.html

另外，既然有很多开源网络框架，那么找一个好的学习下也是很重要的。

这里我学习了陈硕的muduo：https://github.com/chenshuo/muduo，阅读了它的核心源码，这里我推荐他的书籍 “Linux多线程服务端编程” https://book.douban.com/subject/20471211/

# 编码规范

编码规范遵守 “Google C++编码规范” https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/

# cppbox简介

项目构建工具使用cmake，主要目录如下：

```
├── cmake             // 里面是项目相关的cmake files
└── src
    ├── cppbox        // cppbox namespace
    │   ├── log       // 完整的log库
    │   │   └── test
    │   ├── misc      // 零碎的的工具
    │   │   └── test
    │   └── net       // 网络库部分
    │       └── test
    └── http-parser
```

# 一个最简单的服务端程序

摩天大楼都不是一天盖成的，这里我也是从最简单的功能开始，逐步完成。

服务端程序要做的，就是处理客户端的连接，接收请求，发送响应，那么一个最简单的echo程序如下：

```
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstring>
#include <arpa/inet.h>
#include <unistd.h>

#define RW_BUF_LEN 1024

void echo(int connfd) {
  ssize_t n;
  char    buf[RW_BUF_LEN];

  while (true) {
    n = read(connfd, buf, RW_BUF_LEN);
    if (n == 0) {                                  // 对端断开连接
      std::cout << "disconnected" << std::endl;
      break;
    }
    if (n < 0) {                                   // 读取错误
      if (errno != EINTR && errno != EWOULDBLOCK) {
        perror("read error");
        return;
      }
      std::cout << "nonblock" << std::endl;        // 对端没有数据
      sleep(3);
    } else {                                       // 读到数据发送回显
      write(connfd, buf, n);
    }
  }
}

int main() {
  int listenfd = socket(AF_INET, SOCK_STREAM, 0);  // 打开socket
  if (listenfd == -1) {
    perror("listenfd fail");
    return 1;
  }

  struct sockaddr_in serverAddr;
  memset(&serverAddr, 0, sizeof(struct sockaddr_in));

  serverAddr.sin_family = AF_INET;
  if (inet_pton(AF_INET, "127.0.0.1", &(serverAddr.sin_addr)) != 1) {
    std::cout << "inet_pton fail" << std::endl;
    return 1;
  }

//    serverAddr.sin_addr.s_addr = htonl(INADDR_ANY);
  serverAddr.sin_port = htons(8838);

  if (bind(listenfd, (struct sockaddr *) &serverAddr, sizeof(serverAddr)) == -1) {  // bind
    perror("bind fail");
    return 1;
  }

  if (listen(listenfd, 1024) == -1) {      // listen
    perror("listen failed");
    return 1;
  }

  bool running = true;
  while (running) {
    struct sockaddr_in clientAddr;
    memset(&clientAddr, 0, sizeof(struct sockaddr_in));
    socklen_t clientLen;

    int connfd = accept4(listenfd, (struct sockaddr *) &clientAddr, &clientLen, SOCK_NONBLOCK | SOCK_CLOEXEC);    // accept得到connfd，用于之后的连接读写
    if (connfd == -1) {
      running = false;
      perror("accept failed");
    } else {
      echo(connfd);
    }

    close(connfd);
  }

  return 0;
}
```

# 简单说明

上面是一个简单的不能再简单的EchoServer，如果函数使用有不明白的地方，请看下我前面介绍的预备知识。

这个程序，目前只能处理单个客户端的Tcp连接，仅实现了最基本的功能，诸如连接超时、读写buffer等都没有实现，但这个是最基础的东西，之后的一切功能，都是在这个基础上添砖加瓦。

#  cppbox中的封装

cppbox中封装了最常用到的几个基础操作：

`src/cppbox/net/net.h`

```
static const int kDefaultBacklog = 128;

int NewTcpIpV4NonBlockSocket();     // 相当于之后调用accept的sockfd是非阻塞的

misc::ErrorUptr SetSockOpt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

misc::ErrorUptr SetReuseAddr(int sockfd);

misc::ErrorUptr BindForTcpIpV4(int sockfd, const char *ip, uint16_t port);

misc::ErrorUptr Listen(int sockfd, int backlog = kDefaultBacklog);

misc::ErrorUptr BindAndListenForTcpIpV4(int sockfd, const char *ip, uint16_t port, bool reuseaddr = true, int backlog = kDefaultBacklog);  // bind + listen + reuse addr

struct InetAddress {
  std::string ip;
  uint16_t    port;
};

int Accept(int listenfd, struct InetAddress &address, int flags = SOCK_CLOEXEC | SOCK_NONBLOCK); // accept得到非阻塞connfd，同时获得client的ip + port
```

# 使用示例

`src/cppbox/net/test/net_test.cc`

```
TEST_F(NetTest, Misc) {
  int sockfd = ::socket(AF_INET, SOCK_STREAM | SOCK_CLOEXEC, 0);
  if (sockfd == -1) {
    std::cout << cppbox::misc::NewErrorUptrByErrno()->String() << std::endl;
    return;
  }

  auto error_uptr = cppbox::net::BindAndListenForTcpIpV4(sockfd, "127.0.0.1", 8860);
  if (error_uptr != nullptr) {
    std::cout << error_uptr->String() << std::endl;
    return;
  }

  cppbox::net::InetAddress raddr;

  int connfd = cppbox::net::Accept(sockfd, raddr);
  if (connfd == -1) {
    std::cout << "accept error: "
              << cppbox::misc::NewErrorUptrByErrno()->String()
              << std::endl;
    return;
  }

  std::cout << "remote ip is " << raddr.ip << std::endl;
  std::cout << "remote port is " << raddr.port << std::endl;

  ::sleep(10);
}
```
