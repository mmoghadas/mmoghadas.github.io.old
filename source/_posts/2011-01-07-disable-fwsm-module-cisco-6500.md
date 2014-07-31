---
title: Disable FWSM Module Cisco 6500
author: Mike Moghadas
layout: post
categories:
  - Cisco
  - Networking
tags:
  - cisco
  - FWSM
  - Router
---
**To disable the FWSM Module on a Cisco 6500:**

Logon to Router  
{% codeblock lang:objc %}
enable  
hw-module module 1 shutdown  
sh hw-module slot 1 tech-support  
{% endcodeblock %}
