---
title:       "nginx introduce"
subtitle:    ""
description: "总体上介绍nginx的一些设计特点和优点及CLI"
date:        2024-12-30T06:10:16Z
author:      "Wayne"
image:       ""
tags:        ["nginx"]
categories:  ["Tech" ]
---

## Nginx简介overview

nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3代理服务。  
nginx是由俄罗斯的程序设计师Igor Sysoev所开发，使用C语言开发。  
nginx优秀的{{<rawhtml>}}<span style="color:red;">事件驱动(linux的epoll)异步编程模型(架构)，单机能够处理百万级的TCP连接，使它几乎成为高性能的HTTP服务器的代名词</span>{{</rawhtml>}}，这是Nginx的一个最大的特点或长处相较于其它的web server(如apache/lighttpd)。

## Nginx本身的软件框架设计上的特点source design characteristics

* 封装抽象  
  网络异步IO事件 磁盘异步IO事件 定时器异步事件等都被封装抽象成接口，屏蔽隐藏了接口的底层区别，使得上层应用只需要关心接口的实现即可。平台无关(OS/容器等)的功能也被封装了。  
  这个不是啥特色，基本上所有的软件都会使用抽象封装特性。
* 优秀的模块化设计  
  nginx的模块化设计是其一大特色，nginx的模块化设计使得其功能强大而又灵活。  
  这个也不是啥创新，很多软件也都将各个部件模块化设计。这是计算机领域的经典思想，分而治之(二分算法/动态库/静态库等工作思想原理)，通过拆分化繁为简，搭建积木的方式构建大型系统（将各个模块分别独立实现后再整体衔接组装起来）。航空母舰也是通过搭积木的方式将一个一个舱室组装起来的。
* 服务管理友好  
  轻松实现子进程的管理，服务动态升级，配置热加载生效 
* 能够自动选择(性能)最佳的系统调用接口  
  运行在不同的操作系统上，能够自动选择不同系统提供的最佳的系统调用接口，例如在Linux上使用epoll，在FreeBSD上使用kqueue。这样使得nginx在不同的系统上运行都能达到性能最好的状态。


## Nginx的特点(为什么选择使用nginx)Features of Nginx

* 响应快  
  不光是单个请求响应快，在流量高峰期时需要应对处理百万级的TCP连接时，nginx也能够保持快速的响应。
* 高并发支撑  
  能够轻松应对高并发TCP连接且HTTP响应时间依然很小。单机能够支撑10万TCP同时连接。最大可以支撑的并发连接数和内存的大小有关系。
* 高可靠  
  多进程的设计方案，单个worker进程挂掉了不影响全局业务可用性。postgres数据库也是多进程的设计方案，但mysql却是单进程多线程的设计，postgres理论上说应该比mysql更可靠。多进程有天然的故障隔离的好处。
* 低内存开销  
  1万个非活跃的HTTP Keep-Alive连接仅占用2.5MB内存。没有低内存开销的特点不可能实现高并发TCP连接处理的。  
* 支持热加载/热部署  
  多进程的设计，可以在不停止服务的情况下，动态加载配置文件和升级服务。
* 扩展性好  
  nginx的软件架构多模块化低耦合的设计，使得nginx能够轻松扩展新的功能。

## Nginx以性能为王的底层(system call)支撑(在linux上运行)

应对大量TCP连接(高并发)：epoll系统调用是linux platform处理或监控大量fd事件的利器，nginx充分利用了epoll的特性以支持大规模TCP并发连接的处理。  
优化磁盘IO读写效率：使用sendfile系统调用，避免了内核缓冲区拷贝的开销，提示磁盘IO效率。关于sendfile的原理请参考。  


## 优化Kernel的一些配置参数以最大化nginx的运行性能  

kernel会有一些默认的进程相关配置参数，这些参数对于一般的进程运行没有问题，但要想是nginx能够以最大的性能运行，就需要对这些参数进行优化。最明显的一个需要修改的参数是每一个进程运行打开的最大的文件描述符的数量，这个配置决定了nginx能够同时处理的TCP最大连接数。  

`/etc/sysctl.conf` 一些需要或可以修改的配置项如下：  
```
<!-- 进程最大允许打开文件数 -->
fs.max-max = 999999 

<!-- 下面是一些关于TCP/IP协议的配置 -->
net.ipv4.tcp_tw_reuse = 1 // 允许将TIME_WAIT状态的socket重新用于新的连接
net.ipv4.tcp_keepalive_time = 60 // TCP keepalive探测频率。越小能够越快发现已经失效的tcp连接并清理之，能够增加内存的有效利用效率
net.ipv4.tcp_fin_timeout = 30 // server主动close tcp连接时，server保持FIN-WAIT-2状态的时长  
net.ipv4.tcp_max_tw_buckets = 5000 // 最大允许的TIME_WAIT状态的socket数量。过多的TIME_WAIT状态的socket会时web server运行变慢  
net.ipv4.tcp_max_syn_backlog = 1024 // 控制TCP三次握手时，处于SYN_RCVD状态的socket的最大数量,接收SYN请求的队列的最大长度.  
net.ipv4.tcp_rmem = 4096 87380 16777216 // 控制TCP接收缓冲区的大小。TCP的滑动窗口。
net.ipv4.tcp_wmem = 4096 65536 16777216 // 控制TCP发送缓冲区的大小。TCP的滑动窗口。
net.ipv4.tcp_syncookies = 1 // 开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理。应对SYN flood攻击的一种方式。
```

## Nginx CLI(linux)  

nginx默认安装在`/usr/local/nginx`目录下，其可执行文件位于`/usr/local/nginx/sbin/nginx`。 配置文件位于`/usr/local/nginx/conf/nginx.conf`。  

* `/usr/local/nginx/sbin/nginx -g "pid /var/nginx/test.pid;"`启动时指定进程的pid文件位置。  
* `-t`测试配置文件是否正确。
* `-V`显示nginx的编译时.configure指定的参数
* `-s stop`粗暴的快速停止nginx服务，不等待nginx进程处理完当前请求。nginx会先根据pid文件读取进程的pid号，然后向该PID进程发送`SIGTERM`信号，让其停止服务。
* `-s quit`优雅的停止nginx服务，等待nginx进程处理完当前请求后再退出。nginx会先根据pid文件读取进程的pid号，然后向该PID进程发送`SIGQUIT`信号。
* `-s reload`重新加载配置文件，平滑重启nginx服务。nginx会先根据pid文件读取进程的pid号，然后向该PID进程发送`SIGHUP`信号。
* 平滑升级/热部署  
  1. 通知旧版本的nginx进程准备升级了。`kill -s  SIGUSR2 <nginx master pid>`,运行中的旧版本的nginx master进程收到`SIGUSR2`信号后，将会重命名进程的pid文件，这样新版本的nginx进程才能有启动条件。
  2. 启动新版本的nginx进程。
  3. `kill -s SIGQUIT <old nginx master pid>`优雅的关闭老版本的nginx进程。

## 参考references

1. {{<rawhtml>}}<a href="https://blog.clanzx.net/development/listen-backlog.html" target="_blank">监听服务端的socket时的这个C API(<span style="color:red;">int listen(int sockfd, int backlog)</span>)中的backlog参数和<strong>tcp_max_syn_backlog</strong>的关系？</a>{{</rawhtml>}}
2. *深入理解Nginx模块开发与架构解析.pdf*/Chaper1~Chapter2
