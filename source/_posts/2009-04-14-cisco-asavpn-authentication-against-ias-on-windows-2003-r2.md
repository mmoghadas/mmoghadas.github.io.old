---
title: Cisco ASA/VPN authentication against IAS on Windows 2003 R2
author: Mike Moghadas
layout: post
categories:
  - Cisco
  - Networking
  - Windows
tags:
  - ASA
  - cisco
  - IAS
  - PIX
  - Radius
  - Windows 2003
---
**IAS Configuration**

Start > Administrative Tools > Internet Authentication Service  
Create a new Client for ASA Device (192.168.121.254) with Standard Radius configuration  
Configure Remote Access Policies to allow authentication for the above client  
Cisco ASA/PIX

<!--more-->

**Configure ASA to allow authentication via radius server **  
{% codeblock lang:objc %}
aaa-server AuthGrp-PROD protocol radius  
aaa-server AuthGrp-PROD (inside15) host 10.21.80.100 key \***\*****  
Configure Tunnel Group Policy to allow authentication via above radius server  
tunnel-group PRODAdmin general-attributes address-pool PRODAdmin-IPPool  
{% endcodeblock %}
