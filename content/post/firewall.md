---
title: "inbound and outbound firewall rules"
subtitle: ""
description: "Firewalls can support both inbound and outbound firewall rules, but there are important differences between the two. Learn more about each and their uses."
date: 2025-01-02T02:59:07Z
author:      "Wayne"
image: ""
tags: ["inbound", "outbound", "firewall rules"]
categories: ["Tech"]
---

It is critical to compare the roles of inbound and outbound firewall rules before deploying a corporate firewall to ensure it properly secures an enterprise IT environment.

Simply put, inbound firewall rules protect the network from unwanted incoming traffic from the internet or other networks -- in particular, disallowed connections, malware and DoS attacks. Outbound firewall rules control outgoing traffic -- that is, requests to resources outside of the network. For example, a connection request to an email service or the TechTarget website might be allowed, but connection requests to unapproved or dangerous websites are stopped.

A single firewall typically serves both functions, but it's essential to understand the differences -- as well as the benefits and drawbacks -- of inbound and outbound firewall rules.

## Inbound traffic versus outbound traffic

Enterprise networks have both inbound traffic and outbound traffic. {{<rawhtml>}}<strong>Inbound requests originate from outside the network, such as a user with a web browser, email client, server or application making requests -- like FTP and SSH -- or API calls to web services(located inside the network).</strong>{{</rawhtml>}}

{{<rawhtml>}}<strong>Outbound requests originate from inside the network, destined for services on the internet or outside networks.</strong>{{</rawhtml>}}


Inbound traffic originates from outside the network, while outbound traffic originates inside the network.
![Alt text](/img/firewall-inbound-outbound.png)

Firewalls are designed and deployed to prevent inbound traffic from entering a network and to stop outbound traffic from connecting to external resources that are noncompliant with an organization's security policies.

## The difference between inbound and outbound firewall rules

Firewall rules, which are either inbound or outbound, can be customized to allow traffic on specific ports, services and IP addresses to enter or leave the network:

- Inbound firewall rules protect a network by blocking traffic known to be from malicious sources. This stops various attacks, such as malware and DDoS, from affecting internal resources.
- Outbound firewall rules define the traffic allowed to leave a network and reach legitimate destinations. These rules also block requests sent to block requests sent to malicious websites and untrusted domains. They can also prevent data exfiltration by analyzing the contents of emails and files sent from a network.
