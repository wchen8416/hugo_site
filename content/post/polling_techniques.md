---
title: "Polling Techniques"
subtitle: ""
description: "http的4种推送技术：短轮询、长轮询、单向服务器推送（SSE）和 全双工WebSocket"
date: 2025-01-02T07:40:01Z
author:      "Wayne"
image: ""
tags: ["Regular/Short Polling", "Long Polling", "SSE", "WebSocket"]
categories: ["Tech"]
---

## http 的 4 种推送技术

1. 客户端轮询：传统意义上的短轮询（Short Polling）
2. 服务器端轮询：长轮询（Long Polling）
3. 单向服务器推送：Server-Sent Events（SSE）
4. 全双工通信：WebSocket

![Alt text](/img/polling-technique.png)

这个图片里面的 Short Polling 和 Long Polling 的 client side 都是在获取到 server 端发送的 response 之后就 Connection Closed 之后再发起下一次请求,所以它们是短连接短轮询和短连接长轮询。其实不用每一次断开连接也行，变成长连接的短(长)轮询。

## 短轮询和长轮询(Regular(Short) Polling/Long polling)  

![Alt text](/img/long-short-polling.png)

- 长轮询：{{<rawhtml>}}<span style="color:red;">在服务端 hold 住 Http 请求</span>{{</rawhtml>}}（死循环或者 sleep 等等方式），等到目标时间发生，返回 Http 响应,也就是在 response 的内容还没有准备好的情况下故意拖延返回响应消息。优点：在无消息的情况下不会频繁的请求，有消息的情况下会立即返回响应给客户端,缺点：编写复杂 ![Alt text](/img/http-long-polling.png)  
  {{<rawhtml>}}<strong>因此如果浏览器想实时性很高的获取一些数据，可以考虑使用 HTTP 长轮询(这是单工单向通信)或使用 websocket 全双工通信方式。</strong>{{</rawhtml>}}
- 短轮询：重复发送 Http 请求，查询目标事件是否完成，如客户端定时发送 http 请求询问，服务端立即返回成功和失败(由服务端收到查询请求时的实际情况返回成功或失败)，优点：编写简单，缺点：浪费带宽和服务器资源

### 应用

长轮询一般用在 web im, im 实时性要求高, http 长轮询的控制权一直在服务器端, 而数据是在服务器端的, 因此实时性高; 像新浪微薄的 im, 朋友网的 im 以及 webQQ 都是用 http 长轮询实现的; NodeJS 的异步机制貌似可以很好的处理 http 长轮询导致的服务器瓶颈问题, 这个有待研究.

http 短轮询一般用在实时性要求不高的地方, 比如新浪微薄的未读条数查询就是浏览器端每隔一段时间查询的.

## SSE

![Alt text](/img/SSE.png)

## 全双工的 WebSocket 协议通信

TODO

## References

1. {{<rawhtml>}}<a href="https://javascript.info/long-polling" target="_blank">https://javascript.info/long-polling</a>{{</rawhtml>}}
