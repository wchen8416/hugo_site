---
title:       "VMware网络模式工作原理详解"
subtitle:    ""
description: "vmware的三种网络模式及最佳应用场景"
date:        2024-12-29T12:17:40Z
author:      "Wayne"
image:       ""
tags:        ["VMware"]
categories:  ["Tech" ]
---

## VMware网络模式工作原理详解

我的第一台linux系统就是在vmware虚拟机上安装的，目前在工作中我的开发环境也依然是通过VMware搭建的。

notes:  
1. 请注意host机器的网络防火墙设置，如果设置不当,有时间会出现gust无法ping通host的情况，会影响判断。

## VMware三种网络模式

vmware虚拟机主要有三种网络模式：桥接模式、NAT模式和仅主机模式。  
我只使用过nta和bridge模式，仅主机模式(host only)就不介绍了，后面使用到再说。

### NAT模式

使用这种模式，VMware会虚拟出一个局域网同时该局域网还会虚拟出一个DHCP server。所有NAT模式虚拟出来的guests机器都会自动获取到一个虚拟的ip地址(从虚拟出来的dhcp server获取的)，host机器上也会虚拟出来一个虚拟网卡(VMnet8)且它会和这些虚拟出来的guests在同一个网段，这些Guests之间以及guests和host(的VMnet8网卡)之间的网络通信就都是局域网内部通信了(不跨网段了)。这一部分功能解决了guests之间以及guest和host之间的局域网内部通信问题。  

同时VMware还会虚拟出一个NAT设备/服务，所有从guest发往外网的请求都会先被转发到这个虚拟出来的NAT设备上，经过NAPT处理后所有guests机器发出的流量packet的源ip地址会被改成host机器物理网卡的ip，然后再将这个修改后的packet通过host的访问外网的物理网卡发出去。如果在host机器上通过wireshark分析物理网卡的packets,会发现guest发出去的packets的src ip却是host物理网卡的ip而不是guest自己的ip。当guests需要访问外网时，VMware虚拟出来的NAT设备会将所有从guest发往公网的请求转发到host机器的物理网卡上，然后再由host机器的物理网卡将网络流量包packets转发出去，然后再将外网返回的流量包转发给对应的guest。 这一部分功能解决了guests访问外网的问题。

NAT模式的拓扑图如下所示：  
![](/img/VMware-nat-1.png)  

### 桥接模式bridge

使用这种模式时，客户机(guest)必须配置成和宿主机(host)在同一网段，才能和host或公网通信。

bridge时的拓扑图如下所示：  
![](/img/VMware-bridge-1.png)  

实际上和下面的拓扑是一样的：    
![](/img/VMware-bridge-2.png)   

{{<rawhtml>}}
<span style="color:red;">注意：需要正确选择桥接到host的哪个物理网卡</span>，不要host实际使用的是A网卡上网，但配置虚拟机时却桥接到了host的B网卡。(example: 如果host使用wifi上网，则选择桥接到host的wifi网卡guest才可以正常上网通信。)
{{</rawhtml>}}

问题：  
1. VMnet0是什么？
2. guest的虚拟mac和虚拟ip和host机器是平等的？怎么虚拟出来的？  

### host only模式

TODO

## reference

1. {{<rawhtml>}}<a href="https://blog.51cto.com/sddai/3016904" target="_blank">https://blog.51cto.com/sddai/3016904</a>{{</rawhtml>}}
2. {{<rawhtml>}}<a href="https://bbs.huaweicloud.com/blogs/332956" target="_blank">https://bbs.huaweicloud.com/blogs/332956</a>{{</rawhtml>}}