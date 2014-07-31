---
title: Configure Linux server behind F5 Loadbalancer
author: Mike Moghadas
layout: post
categories:
  - Linux
  - Networking
tags:
  - F5
  - Linux
---
Adding Networking: VLANS, ROUTES AND ROUTE DOMAINS  
Create VLAN  
Name: VLAN160-net1-0  
Tag: 160  
Interfaces: Trunk-1

Create Route Domain  
ID: 160  
VLANs: VLAN160-net1-0

<!--more-->

Create Self IP address(s)  
Self IPs > Create  
IP Addres: 10.176.160.30%160  
Netmask: 255.255.255.0  
VLAN: VLAN160-net1-0

Self IPs > Create  
IP Addres: 10.176.160.40%160  
Netmask: 255.255.255.0  
VLAN: VLAN160-net1-0  
Floating IP: CHECK

Create Routes  
Route Domain ID: 160  
Gateway Address: 10.176.160.1%160

Adding Virtual Server (LB101) for the servers 

Create Monitor  
Name: MON-NET-8080  
Type: TCP  
Service Port: 8080

Create a new Node  
Local Traffic > Nodes > Create  
Address: 10.176.160.103%160  
Name: NETAPP101  
Health Monitors > Node Specific > icmp  
Repeat for NETAPP102(10.176.160.104%160)

Create a new Pool  
Local Traffic > Pools > Create  
Name: POOL-NET  
Health Monitors: gateway_icmp, MON-NET-8080  
Action on Service Down: Reselect  
Load Balancing Method: Round Robin  
Members: NETAPP101, NETAPP102  
Service Port: 8080

Create a Profile  
Local Traffic > Profiles > Protocols > FastL4 > Create  
Name: NET-FastL4  
Custom > Loose Close > Enabled

Create a new Virtual Server  
Local Traffic > Virtual Servers > Create  
Name: VS-NET-SSL  
Destination: Host/10.176.160.51%160  
Service Port: 8080  
Type: Performance (Layer4)  
Protocol: TCP  
Protocol Profile (Client): NET-FastL4  
VLAN Traffic: VLAN160-net1-0  
Address Translation: Disabled  
Port Translation: Disabled  
SNAT Pool: NONE  
Default Pool: POOL-NET

ON NETAPP101 and NETAPP102

\# This is required to get this ports to work with load balancer  
\# iptables -t nat -A PREROUTING -d LOADBALANCEADDRESSHERE -j DNAT &#8211;to-dest SERVERADDRESSHERE  
{% codeblock lang:objc %}
$ iptables -t nat -A PREROUTING -d 10.176.160.51 -j DNAT &#8211;to-dest 10.176.160.103  
{% endcodeblock %}

\# Save config to file  
{% codeblock lang:objc %}
/etc/init.d/iptables save  
{% endcodeblock %}

\# List the newly created NAT rule  
{% codeblock lang:objc %}
$ iptables -t nat -L  
{% endcodeblock %}
