---
title: Django, WSGI, Apache2 on Ubuntu 10.04 LTS
author: Mike Moghadas
layout: post
categories:
  - Linux
  - Web Server
tags:
  - apache
  - Django
  - wsgi
---
Install Apache2  
{% codeblock lang:objc %}
sudo apt-get install apache2  
{% endcodeblock %}

<!--more-->

Install MOD_WSGI  
{% codeblock lang:objc %}
sudo apt-get install libapache2-mod-wsgi  
{% endcodeblock %}

Install Python (Installed by default, but just in case)  
{% codeblock lang:objc %}
sudo apt-get install python  
{% endcodeblock %}

Install Python Setup Tools  
{% codeblock lang:objc %}
sudo apt-get install python-setuptools  
{% endcodeblock %}

Install Django  
{% codeblock lang:objc %}
apt-get install python-pip  
pip install django  
{% endcodeblock %}
