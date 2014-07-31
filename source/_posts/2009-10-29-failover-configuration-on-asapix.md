---
title: Failover Configuration on ASA/PIX
author: Mike Moghadas
layout: post
categories:
  - Cisco
  - Networking
tags:
  - active
  - Active/Standby
  - ASA
  - firewall
  - PIX
  - Standby
---
**To configure ASA/PIX redundancy:**

<!--more-->

**Active Unit**

{% codeblock lang:objc %}
interface Ethernet0/0  
ip address 63.251.250.86 255.255.255.0 standby 63.251.250.87  
interface Ethernet0/1  
ip address 10.176.80.101 255.255.255.0 standby 10.176.80.102  
interface Ethernet0/3  
description LAN/STATE Failover Interface  
no shut  
failover lan unit primary  
failover lan interface failover Ethernet0/3  
failover key \***\***  
failover link failover Ethernet0/3  
failover interface ip failover 192.168.98.1 255.255.255.0 standby 192.168.98.2  
failover  
{% endcodeblock %}

**Failover Unit**  
{% codeblock lang:objc %}
failover lan unit secondary  
failover lan interface failover Ethernet0/3  
failover key \***\***  
failover interface ip failover 192.168.98.1 255.255.255.0 standby 192.168.98.2  
failover  
{% endcodeblock %}
