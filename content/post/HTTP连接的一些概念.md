---
title: "HTTP连接类型"
subtitle: ""
description: "HTTP协议中的短轮询 长轮询 长连接 短连接。HTTP header的keep-alive。"
date: 2024-12-30T13:38:09Z
author:      "Wayne"
image: ""
tags:
  [
    "HTTP",
    "长连接 短连接",
    "短轮询 长轮询",
    "Regular Polling",
    "Long Polling",
    "http keep-alive",
  ]
categories: ["Tech"]
---

## HTTP 短连接&HTTP 长连接

![Alt text](/img/HTTP连接类型.png)

在长连接的应用场景下，client 端一般不会主动关闭它们之间的连接，Client 与 server 之间的连接如果一直不关闭的话，会存在一个问题，随着客户端连接越来越多，server 早晚有扛不住的时候，这时候 server 端需要采取一些策略，如关闭一些长时间没有读写事件发生的连接，这样可以避免一些恶意连接导致 server 端服务受损；还可以以客户端机器为颗粒度，限制每个客户端的最大长连接数，这样可以完全避免某个蛋疼的客户端连累后端服务。长连接和短连接的产生在于 client 和 server 采取的关闭策略，具体的应用场景采用具体的策略，没有十全十美的选择，只有合适的选择。

### 应用场景区别：

- 一般长连接（追求实时性高的场景）用于少数 client-end to server-end 的频繁的通信，例如：数据库的连接用长连接， 如果用短连接频繁的通信会造成 socket 错误，而且频繁的 socket 创建也是对资源的浪费。
- 而像 WEB 网站的 http 服务一般都用短链接（追求资源易回收场景），因为长连接对于服务端来说会耗费一定的资源，而像 WEB 网站这么频繁的成千上万甚至上亿客户端的连接用短连接会更省一些资源。

## 短轮询和长轮询

- 长轮询：{{<rawhtml>}}<span style="color:red;">在服务端 hold 住 Http 请求</span>{{</rawhtml>}}（死循环或者 sleep 等等方式），等到目标时间发生，返回 Http 响应,也就是在 response 的内容还没有准备好的情况下故意拖延返回响应消息。优点：在无消息的情况下不会频繁的请求，有消息的情况下会立即返回响应给客户端,缺点：编写复杂
- 短轮询：重复发送 Http 请求，查询目标事件是否完成，如客户端定时发送 http 请求询问，服务端立即返回成功和失败(由服务端收到查询请求时的实际情况返回成功或失败)，优点：编写简单，缺点：浪费带宽和服务器资源

### 应用

长轮询一般用在 web im, im 实时性要求高, http 长轮询的控制权一直在服务器端, 而数据是在服务器端的, 因此实时性高; 像新浪微薄的 im, 朋友网的 im 以及 webQQ 都是用 http 长轮询实现的; NodeJS 的异步机制貌似可以很好的处理 http 长轮询导致的服务器瓶颈问题, 这个有待研究.

http 短轮询一般用在实时性要求不高的地方, 比如新浪微薄的未读条数查询就是浏览器端每隔一段时间查询的.

## References

1. {{<rawhtml>}}<a href="https://javascript.info/long-polling" target="_blank">https://javascript.info/long-polling</a>{{</rawhtml>}}
