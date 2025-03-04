---
title: "packebeat的fanout_group原理研究"
subtitle: " "
description: " "
date: 2025-02-23T08:11:53Z
author:      "Wayne"
image: ""
tags: ["packetbeat", "af_packet", "fanout"]
categories: ["Tech"]
---

## packetbeat 官网 fanout_group 功能介绍

"To scale processing across multiple Packetbeat processes, a fanout group identifier can be specified. When fanout_group is used {{<rawhtml>}}<span  style="color:red;">the Linux kernel splits packets across Packetbeat instances in the same group by using a flow hash. It computes the flow hash modulo with the number of Packetbeat processes in order to consistently route flows to the same Packetbeat instance.</span>{{</rawhtml>}}  
This is only available on Linux and requires using type: `af_packet`. Each process must be running in same network namespace. All processes must use the same interface settings. You must take responsibility for running multiple instances of Packetbeat."

## Refences

1. https://man7.org/linux/man-pages/man7/packet.7.html
2. https://www.kernel.org/doc/html/latest/networking/packet_mmap.html#packet-fanout
3. https://csulrong.github.io/blogs/2022/03/10/linux-afpacket/