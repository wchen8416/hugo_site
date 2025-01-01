---
title: "KeepAlive"
subtitle: ""
description: "TCP/IP中的keepalive和HTTP中的keep-alive"
date: 2024-12-30T13:28:02Z
author:      "Wayne"
image: ""
tags: ["setsockopt", "TCP KeepAlive", "HTTP Keep-Alive"]
categories: ["Tech"]
---

## TCP KeepAlive 机制简介

TCP 长连接下，客户端和服务器若长时间无数据交互情况下，若一方出现异常情况关闭连接，抑或是连接中间路由出于某种机制断开连接，而此时另一方不知道对方状态而一直维护连接，浪费系统资源的同时，也会引起下次数据交互时出错。

为了解决此问题，引入了 TCP KeepAlive 机制（并非标准规范，但操作系统一旦实现，默认情况下须为关闭，可以被上层应用开启和关闭）。其基本原理是在此机制开启时，当长连接无数据交互一定时间间隔时，连接的一方会向对方发送保活探测包，如连接仍正常，对方将对此确认回应。

关于 TCP KeepAlive 机制的详细背景可以参考《TCP/IP 详解 卷 1:协议》一书。

### TCP KeepAlive 设置参数(三个)

- tcp_keepalive_time (integer; default: 7200; since Linux 2.2)
  The number of seconds a connection needs to be idle before TCP begins sending out keep-alive probes. Keep-alives are sent only when the SO_KEEPALIVE socket option is enabled. The default value is 7200 seconds (2 hours). An idle connection is terminated after approximately an additional 11 minutes (9 probes an interval of 75 seconds apart) when keep-alive is enabled. 在 TCP 保活打开的情况下，最后一次数据交换到 TCP 发送第一个保活探测包的间隔，即允许的持续空闲时长，或者说每次正常发送心跳的周期，默认值为 7200s（2h）。
- tcp_keepalive_intvl (integer; default: 75; since Linux 2.2)  
  The number of seconds between individual keep-alive probes. The default value is 75 seconds. 在 tcp_keepalive_time 之后，没有接收到对方确认，继续发送保活探测包的发送频率，默认值为 75s。
- tcp_keepalive_probes (integer; default: 9; since Linux 2.2)  
   The maximum number of TCP keep-alive probes to send before giving up and killing the connection if no response is obtained from the other end. 在 tcp_keepalive_time 之后，最大允许发送保活探测包的次数，到达此次数后直接放弃尝试，并关闭连接，默认值为 9（次）。

在 Linux 系统中，可以通过 sysctl 来设置这三个参数。例如：

```
sudo sysctl -w net.ipv4.tcp_keepalive_time=180
sudo sysctl -w net.ipv4.tcp_keepalive_intvl=30
sudo sysctl -w net.ipv4.tcp_keepalive_probes=5
```

### TCP KeepAlive 报文格式

{{<rawhtml>}}<strong>TCP KeepAlive 探测报文是一种没有任何数据，同时 ACK 标志被置上的报文，报文中的序列号为上次发生数据交互时 TCP 报文序列号减 1。</strong>{{</rawhtml>}}比如上次本端和对端数据交互的最后时刻，对端回应给本端的 ACK 报文序列号为 N（即下次本端向对端发送数据，序列号应该为 N），则本端向对端发送的保活探测报文序列号应该为 N-1。

![alt text](/img/tcp-keepalive-wireshark.png)

### TCP KeepAlive 编程中的应用

- java socket 编程中有个 keepalive 选项
  java socket 编程中有个 keepalive 选项，看到这个选项经常会误解为长连接，不设置则为短连接，实则不然。
  socket 连接建立之后，只要双方均未主动关闭连接，那这个连接就是会一直保持的，就是持久的连接。keepalive 只是为了防止连接的双方发生意外而通知不到对方，导致一方还持有连接，占用资源。
  其实这个选项的意思是 TCP 连接空闲时是否需要向对方发送探测包，实际上是依赖于底层的 TCP 模块实现的，java 中只能设置是否开启，不能设置其详细参数，只能依赖于系统配置。

  首先看看源码里面是怎么说的
  {{<rawhtml>}}
  <div style="text-align: left;">
    <img src="/img/java-socket-keepalive.png" alt="">
  </div>
  {{</rawhtml>}}

- C 语言 socket 设置
  对于 Socket 而言，可以在程序中通过 socket 选项开启 TCP KeepAlive 功能，并配置参数。对应的 Socket 选项分别为 `SO_KEEPALIVE`、`TCP_KEEPIDLE`、`TCP_KEEPCNT`、`TCP_KEEPINTVL`。

  ```c
  int keepalive = 1;          // 开启TCP KeepAlive功能
  int keepidle = 7200;        // tcp_keepalive_time
  int keepcnt = 9;            // tcp_keepalive_probes
  int keepintvl = 75;         // tcp_keepalive_intvl

  setsockopt(socketfd, SOL_SOCKET, SO_KEEPALIVE, (void *)&keepalive, sizeof(keepalive));
  setsockopt(socketfd, SOL_TCP, TCP_KEEPIDLE, (void *) &keepidle, sizeof (keepidle));
  setsockopt(socketfd, SOL_TCP, TCP_KEEPCNT, (void *)&keepcnt, sizeof (keepcnt));
  setsockopt(socketfd, SOL_TCP, TCP_KEEPINTVL, (void *)&keepintvl, sizeof (keepintvl));
  ```

## HTTP Keep-Alive

HTTP Keep-Alive 是 HTTP/1.1 协议中的一个特性，它允许在单个 TCP 连接上发送和接收多个请求或响应。这意味着客户端和服务器可以在一个持久连接的上下文中多次交换数据，而不需要为每个单独的请求建立新的连接。

### 如何开启 HTTP Keep-Alive

在 HTTP/1.1 中，默认情况下是启用 Keep-Alive 的。如果要显式地指定使用 Keep-Alive，可以在 HTTP 头部中添加以下字段：

```http
Connection: keep-alive
Keep-Alive: timeout=5, max=1000
```

其中 `timeout` 表示保持连接的时间（单位通常是秒），`max` 表示在一个 Keep-Alive 会话中可以重用的最大请求数。如果浏览器不支持这些参数，它将忽略它们并继续使用默认值。

## HTTP Keep-Alive 和 TCP KeepAlive 的区别

HTTP Keep-Alive 指的是 HTTP/1.1 协议中，通过在请求头或响应头中设置 `Connection: keep-alive` 来保持连接持续打开状态，以便后续的多个请求可以复用同一个 TCP 连接。

而 TCP KeepAlive 是指在传输层（TCP）上的一种机制，用于检测一个空闲连接的另一端是否仍然可达，以避免因网络故障或其他原因导致的半开连接占用资源。它并不直接与 HTTP 的持久连接特性相关联，但可以在使用 HTTP Keep-Alive 时发挥作用，确保即使在长时间无数据传输的情况下，连接也不会被意外关闭。

### 如何理解二者关系

- **底层支持**：{{<rawhtml>}}<span style="color:red;">HTTP Keep-Alive 需要依赖于底层的 TCP KeepAlive 功能来维持长连接。如果没有启用 TCP KeepAlive，即使设置了 HTTP Keep-Alive，一旦连接空闲一段时间后，对方也可能因为超时自动断开连接。</span>{{</rawhtml>}}

- **配置方式**：虽然两者都与“KeepAlive”命名相似，但在不同的层次上有不同的配置方式和参数。HTTP Keep-Alive 主要通过 HTTP 头部字段控制，而 TCP KeepAlive 则通常是通过操作系统级别的配置或者 socket 选项进行设置。

- **目的不同**：{{<rawhtml>}}<span style="color:red;">HTTP Keep-Alive 的主要目的是为了减少建立和关闭连接的开销，提高资源利用率；而 TCP KeepAlive 主要是为了保证网络连接的可靠性，防止因各种原因导致的半开或全开连接占用资源。</span>{{</rawhtml>}}
