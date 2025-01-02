---
title: "各种保活(KeepAlive)机制研究"
subtitle: ""
description: "TCP/IP(socket)中的keepalive。HTTP header中的keep-alive。以及应用层自己添加的heartbeat心跳机制的研究。"
date: 2024-12-30T13:28:02Z
author:      "Wayne"
image: ""
tags: ["setsockopt", "TCP KeepAlive", "HTTP Keep-Alive", "应用层心跳heartbeat"]
categories: ["Tech"]
---

在计算机领域中，经常会遇到各种保活机制，比如 TCP KeepAlive 机制、HTTP Keep-Alive 以及应用层的心跳（heartbeat）机制。这些保活机制在维护网络连接、检测资源状态等方面起着重要作用。本文将介绍这些保活机制的基本原理、编程中的应用以及一些注意事项。

## TCP KeepAlive 机制简介

TCP 长连接下，客户端和服务器若长时间无数据交互情况下，若一方出现异常情况关闭连接，抑或是连接中间路由出于某种机制断开连接，而此时另一方不知道对方状态而一直维护连接，浪费系统资源的同时，也会引起下次数据交互时出错。

为了解决此问题，引入了 TCP KeepAlive 机制（并非标准规范，但操作系统一旦实现，默认情况下须为关闭，可以被上层应用开启和关闭）。{{<rawhtml>}}<strong>其基本原理是在此机制开启时，当长连接无数据交互一定时间间隔时，连接的一方会向对方发送保活探测包，如连接仍正常，对方将对此确认回应。</strong>{{</rawhtml>}}

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
  看到这个选项经常会误解为长连接，不设置则为短连接，实则不然。
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

## HTTP 之 keep-alive

HTTP Keep-Alive 是 HTTP/1.1 协议中的一个特性，它允许在单个 TCP 连接上发送和接收多个请求或响应。这意味着客户端和服务器可以在一个持久连接的上下文中多次交换数据，而不需要为每个单独的请求建立新的连接。

### 如何开启 HTTP Keep-Alive

在 HTTP/1.1 中，默认情况下是启用 Keep-Alive 的。如果要显式地指定使用 Keep-Alive，可以在 HTTP 头部中添加以下字段：

```http
Connection: keep-alive
Keep-Alive: timeout=5, max=1000
```

其中 `timeout` 表示保持连接的时间（单位通常是秒），`max` 表示在一个 Keep-Alive 会话中可以重用的最大请求数。如果浏览器不支持这些参数，它将忽略它们并继续使用默认值。

### HTTP keep-alive 和 TCP KeepAlive 的区别与联系

HTTP Keep-Alive 指的是 HTTP/1.1 协议中，通过在请求头或响应头中设置 `Connection: keep-alive` 来保持连接持续打开状态，以便后续的多个请求可以复用同一个 TCP 连接。

而 TCP KeepAlive 是指在传输层（TCP）上的一种机制，用于检测一个空闲连接的另一端是否仍然可达，以避免因网络故障或其他原因导致的半开连接占用资源。它并不直接与 HTTP 的持久连接特性相关联，但可以在使用 HTTP Keep-Alive 时发挥作用，确保即使在长时间无数据传输的情况下，连接也不会被意外关闭。

#### 如何理解二者关系

- **底层支持**：{{<rawhtml>}}<span style="color:red;">HTTP Keep-Alive 需要依赖于底层的 TCP KeepAlive 功能来维持长连接。如果没有启用 TCP KeepAlive，即使设置了 HTTP Keep-Alive，一旦连接空闲一段时间后，对方也可能因为超时自动断开连接。</span>{{</rawhtml>}}

- **配置方式**：虽然两者都与“KeepAlive”命名相似，但在不同的层次上有不同的配置方式和参数。HTTP Keep-Alive 主要通过 HTTP 头部字段控制，而 TCP KeepAlive 则通常是通过操作系统级别的配置或者 socket 选项进行设置。

- **目的不同**：{{<rawhtml>}}<span style="color:red;">HTTP Keep-Alive 的主要目的是为了减少建立和关闭连接的开销，提高资源利用率；而 TCP KeepAlive 主要是为了保证网络连接的可靠性，防止因各种原因导致的半开或全开连接占用资源。</span>{{</rawhtml>}}

## 应用层心跳保活机制 heartbeat(这里以 IM 即时通讯系统设计中的心跳机制为例进行说明)

在使用 TCP 长连接的 IM 服务设计中，往往都会涉及到心跳。心跳一般是指某端(绝大多数情况下是客户端)每隔一定时间向对端发送自定义指令，以判断双方是否存活，因其按照一定间隔发送，类似于心跳，故被称为心跳指令。

### TCP 协议不是自带 KeepAlive 的吗？为什么还需要在应用层再增加一个心跳机制？

那么问题就随之而来了：为什么需要在应用层做心跳，难道 TCP 不是个可靠连接吗？我们不能够依赖 TCP 做断线检测吗？比如使用 TCP 的 KeepAlive 机制来实现。应用层心跳是目前的最佳实践吗？怎么样的心跳才是最佳实践。

### 在 IM 系统中保持有效长连接的重要性

对于客户端而言，使用 TCP 长连接来实现业务的最大驱动力在于：在当前连接可用的情况下，每一次请求都只是简单的数据发送和接受，免去了 DNS 解析，连接建立等时间，大大加快了请求的速度，同时也有利于接受服务器的实时消息。但前提是连接可用。

如果连接无法很好地保持，每次请求就会变成撞大运：运气好，通过长连接发送请求并收到反馈。运气差，当前连接已失效，请求迟迟没有收到反馈直到超时，又需要一次连接建立的过程，其效率甚至还不如 HTTP。而连接保持的前提必然是检测连接的可用性，并在连接不可用时主动放弃当前连接并建立新的连接。

基于这个前提，必须要有一种机制用于检测连接可用性。同时移动网络的特殊性也要求客户端需要在空余时间发送一定的信令，避免连接被回收。详见微信和运营商的撕 B（另一篇针对微信的信令风暴技术研究文章请见：《微信对网络影响的技术试验及分析》）。

而对于服务器而言，能够及时获悉连接可用性也非常重要：一方面服务器需要及时清理无效连接以减轻负载，另一方面也是业务的需求，如游戏副本中服务器需要及时处理玩家掉线带来的问题。

### TCP 的 KeepAlive 无法替代应用层心跳保活 heartbeat 机制的原因

上面说了保持连接的重要性，那么现在回到具体实现上。为什么我们需要使用应用层心跳来做检测，而不是直接使用 TCP 的特性呢？  
我们知道 TCP 是一个基于连接的协议，其连接状态是由一个状态机进行维护，连接完毕后，双方都会处于 established 状态，这之后的状态并不会主动进行变化。这意味着如果上层不进行任何调用，一直使 TCP 连接空闲，那么这个连接虽然没有任何数据，但仍是保持连接状态，一天、一星期、甚至一个月，即使在这期间中间路由崩溃重启无数次。举个现实中经常遇到的栗子：当我们 ssh 到自己的 VPS 上，然后不小心踢掉网线，此时的网络变化并不会被 TCP 检测出，当我们重新插回网线，仍旧可以正常使用 ssh，同时此时并没有发生任何 TCP 的重连。

有人会说 TCP 不是有 KeepAlive 机制么，通过这个机制来实现不就可以了吗？但是事实上，TCP KeepAlive 的机制其实并不适用于此。Keep Alive 机制开启后，TCP 层将在定时时间到后发送相应的 KeepAlive 探针以确定连接可用性。一般时间为 7200 s（详情请参见《TCP/IP 详解》中第 23 章），失败后重试 10 次，每次超时时间 75 s。显然默认值无法满足我们的需求，而修改过设置后就可以满足了吗？答案仍旧是否定的。

因为 TCP KeepAlive 是用于检测连接的死活，而心跳机制则附带一个额外的功能：检测通讯双方的存活状态。两者听起来似乎是一个意思，但实际上却大相径庭。

考虑一种情况，某台服务器因为某些原因导致负载超高，CPU 100%，无法响应任何业务请求，但是使用 TCP 探针则仍旧能够确定连接状态，{{<rawhtml>}}<span style="color:red;">这就是典型的连接活着但业务提供方已死的状态</span>{{</rawhtml>}}，对客户端而言，这时的最好选择就是断线后重新连接其他服务器，而不是一直认为当前服务器是可用状态，一直向当前服务器发送些必然会失败的请求。

从上面我们可以知道，KeepAlive 并不适用于检测双方存活的场景，这种场景还得依赖于应用层的心跳。应用层心跳有着更大的灵活性，可以控制检测时机，间隔和处理流程，甚至可以在心跳包上附带额外信息。从这个角度而言，应用层的心跳的确是最佳实践。

### 应用层心跳保活 heartbeat 机制的实现方案

从上面我们可以得出结论，目前而言，应用层心跳的确是检测连接有效性，双方是否存活的最佳实践，那么剩下的问题就是怎么实现。

最简单粗暴做法当然是定时心跳，如每隔 30 秒心跳一次，15 秒内没有收到心跳回包则认为当前连接已失效，断开连接并进行重连。这种做法最直接，实现也简单。唯一的问题是比较耗电和耗流量。以一个协议包 5 个字节计算，一天收发 2880 个心跳包，一个月就是 5 _ 2 _ 2880 \* 30 = 0.8 M 的流量，如果手机上多装几个 IM 软件，每个月光心跳就好几兆流量没了，更不用说频繁的心跳带来的电量损耗。

既然频繁心跳会带来耗电和耗流量的弊端，改进的方向自然是减少心跳频率，但也不能过于影响连接检测的实时性。基于这个需求，一般可以将心跳间隔根据程序状态进行调整，当程序在后台时(这里主要考虑安卓)，尽量拉长心跳间隔，5 分钟、甚至 10 分钟都可以。

而当 App 在前台时则按照原来规则操作。连接可靠性的判断也可以放宽，避免一次心跳超时就认为连接无效的情况，使用错误积累，只在心跳超时 n 次后才判定当前连接不可用。当然还有一些小 trick 比如从收到的最后一个指令包进行心跳包周期计时而不是固定时间，这样也能够一定程度减少心跳次数。

## References

1. {{<rawhtml>}}<a href="http://www.52im.net/thread-281-1-1.html" target="_blank">http://www.52im.net/thread-281-1-1.html</a>{{</rawhtml>}}
2. {{<rawhtml>}}<a href="https://www.cnblogs.com/hueyxu/p/15759819.html" target="_blank">https://www.cnblogs.com/hueyxu/p/15759819.html</a>{{</rawhtml>}}
