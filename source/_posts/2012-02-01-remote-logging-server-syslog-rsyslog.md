---
title: 'Remote Logging Server: Syslog | Rsyslog'
author: Mike Moghadas
layout: post
categories:
  - Linux
---
Enabling remote logging on in linux is very simple. Most distributions include Syslog and most recently Rsyslog. The configuration is different between these two products, but can get either excepting remote logs by editing a single file. You should always read the man pages and read about syslog security while at it.

<!--more-->

Syslog  
\# Add &#8220;-r&#8221; to the end of the &#8220;RSYSLOGD_OPTIONS&#8221;. Example:  
{% codeblock lang:objc %}
vi /etc/default/syslog  
RSYSLOGD_OPTIONS=&#8221;-c3 -r&#8221;

service syslog restart  
{% endcodeblock %}

Rsyslog  
\# Uncomment the following lines in /etc/rsyslog.conf file  
{% codeblock lang:objc %}
$ModLoad imudp  
$UDPServerRun 514

service rsyslog restart  
{% endcodeblock %}
