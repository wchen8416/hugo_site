---
title: "netmap"
subtitle: ""
description: 'netmap is a framework for high speed packet I/O. Together with its companion VALE software switch, it is implemented as a single kernel module and available for FreeBSD, Linux and now also Windows (OSX still missing, unfortunately). netmap supports access to network cards (NICs), host stack, virtual ports (the "VALE" switch), and "netmap pipes". It can easily reach line rate on 10G NICs (14.88 Mpps), over 30 Mpps on 40G NICs (limited by the NIC''s hardware), over 20 Mpps on VALE ports, and over 100 Mpps on netmap pipes. There is netmap support for QEMU, libpcap (hence, all libpcap applications can use it), the bhyve hypervisor, the Click modular ruter, and a number of other applications.'
date: 2025-01-02T14:01:37Z
author:      "Wayne"
image: ""
tags: ["packet I/O"]
categories: ["Tech"]
---

## Reference

{{<rawhtml>}}<a href="http://info.iet.unipi.it/~luigi/netmap/" target="_blank">http://info.iet.unipi.it/~luigi/netmap/</a>{{</rawhtml>}}
