---
title: 'F5 Failover: Active/Standby Issue'
author: Mike Moghadas
layout: post
categories:
  - Networking
tags:
  - Active/Standby
  - F5
  - Failover
---
In an F5 Active/Standby setup, Unit1 takes over as ACTIVE when it recovers from an intentional reboot or power off

**Problem:** This causes loss of network access to the VIPs due to invalid ARP (Since the Standby unit has taken over)

**Solution:** SSH to both devices and…

<!--more-->

On Active Unit (Unit1)  
{% codeblock lang:objc %}
\# b db failover.forceactive show  
Failover.ForceActive = enable  
\# b db failover.forceactive disable  
\# b db failover.forceactive show  
Failover.ForceActive = disable  
{% endcodeblock %}

On Standby Unit (Unit2)  
{% codeblock lang:objc %}
\# b db failover.forcestandby show  
Failover.ForceStandby = enable  
\# b db failover.forcestandby disable  
\# b db failover.forcestandby show  
Failover.ForceStandby = disable  
{% endcodeblock %}
