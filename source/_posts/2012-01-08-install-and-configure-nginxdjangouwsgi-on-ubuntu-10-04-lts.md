---
title: Install and Configure Nginx+Django+uWSGI on Ubuntu 10.04 LTS
author: Mike Moghadas
layout: post
categories:
  - Linux
  - Web Server
tags:
  - Django
  - memcached
  - Nginx
  - uWSGI
---
Getting Nginx+Django+uWSGI to work together via repositories is fairly easy.  The same goes for getting this setup working from source.. You just have to have good instructions. Here&#8217;s a very simple and quick install guide.

Install packages required to build and install from source. Also, Nginx is installed to take care of some of the prerequisites and to install configuration files.

<!--more-->

{% codeblock lang:objc %}
apt-get install build-essential supervisor python python-dev python-setuptools python-pip nginx libxml2-dev libpcre3 libpcre3-dev unzip  
{% endcodeblock %}

Remove and purge Nginx installation. Configuration and startup files are left behind.  
{% codeblock lang:objc %}
apt-get remove nginx  
apt-get purge nginx  
{% endcodeblock %}

Install Django  
{% codeblock lang:objc %}
pip install django  
{% endcodeblock %}

If using memcached, Install the memcached client  
{% codeblock lang:objc %}
easy_install python-memcached  
{% endcodeblock %}

Create the install structure and download/extract the required files  
{% codeblock lang:objc %}
mkdir /root/installs  
cd /root/installs  
wget http://projects.unbit.it/downloads/uwsgi-0.9.9.2.tar.gz  
wget http://nginx.org/download/nginx-1.0.10.tar.gz  
tar zxvf nginx-1.0.10.tar.gz  
tar zxvf uwsgi-0.9.9.2.tar.gz  
{% endcodeblock %}

Build and Install uWSGI  
{% codeblock lang:objc %}
cd uwsgi-0.9.9.2  
make  
cp uwsgi /usr/sbin/  
{% endcodeblock %}

Some preperations to before building Nginx  
{% codeblock lang:objc %}
mkdir /var/log/nginx  
touch /var/log/nginx/error.log  
touch /var/log/nginx/access.log  
{% endcodeblock %}

Build and Install Nginx  
{% codeblock lang:objc %}
cd ../nginx-1.0.10  
./configure &#8211;conf-path=/etc/nginx/nginx.conf \  
&#8211;sbin-path=/usr/sbin \  
&#8211;error-log-path=/var/log/nginx/error.log  
make  
make install  
{% endcodeblock %}

Create structure and build your DEMO Django project  
{% codeblock lang:objc %}
mkdir -p /var/www/mobile  
cd /var/www/mobile  
django-admin.py startproject MobileTicketSales  
cd MobileTicketSales  
vi wsgi_app.py  
#! /usr/bin/env python  
import sys  
import os  
import django.core.handlers.wsgi  
sys.path.append(&#8216;/var/www/mobile/&#8217;)  
os.environ['DJANGO\_SETTINGS\_MODULE'] = &#8216;MobileTicketSales.settings&#8217;  
application = django.core.handlers.wsgi.WSGIHandler()  
{% endcodeblock %}

Configure init daemon to control your uWSGI application  
{% codeblock lang:objc %}
vi /etc/init/mobile.conf  
description &#8220;uWSGI server for Project Foo&#8221;  
start on runlevel [2345]  
stop on runlevel [!2345]  
respawn  
exec /usr/sbin/uwsgi  
&#8211;socket /var/run/mobile.sock  
&#8211;chmod-socket  
&#8211;module wsgi_app  
&#8211;pythonpath /var/www/mobile/MobileTicketSales  
-p 1  
{% endcodeblock %}

Start your uWSGI  
{% codeblock lang:objc %}
initctl reload-configuration  
service mobile start  
{% endcodeblock %}

Configure Nginx to start your Site  
{% codeblock lang:objc %}
cd /etc/nginx  
cp nginx.conf nginx.conf.orig  
vi nginx.conf  
\# include /etc/nginx/conf.d/*.conf;  
include /etc/nginx/sites-enabled/*.conf;  
For additional security add the following to your nginx.conf  
cd /etc/nginx  
cp nginx.conf nginx.conf.orig  
vi nginx.conf  
{% endcodeblock %}

\# You may also add the below section to secure Nginx a bit  
\# NOTE: Nginx default keepalive_timeout might currently be set. Comment out the default and use the below setting.

{% codeblock lang:objc %}
########################################################  
\# Security Directives #  
########################################################  
#Start: Size Limits & Buffer Overflows  
client\_body\_buffer_size 1K;  
client\_header\_buffer_size 1k;  
client\_max\_body_size 1k;  
large\_client\_header_buffers 2 1k;

#Start: Timeouts  
client\_body\_timeout 10;  
client\_header\_timeout 10;  
keepalive_timeout 5 5;  
send_timeout 10;

#Store session states in zone slimits &#8211; 32,000 sessions with 32 bytes/session/1m  
limit\_zone slimits $binary\_remote_addr 5m;

\# Restricts connections from a single ip address  
limit_conn slimits 4;  
}  
{% endcodeblock %}

Create some configuration directories. This is where you drop Site files  
\# Make sure the following directories exist. If not, create them  
{% codeblock lang:objc %}
mkdir sites-available  
mkdir sites-enabled  
{% endcodeblock %}

Remove the default Site if you like. If you plan to leave this site alone, you must configure your uWSGI applicaiton to run on a alternative port  
{% codeblock lang:objc %}
rm sites-enabled/default  
{% endcodeblock %}

Create your Site (VirtualHost)  
{% codeblock lang:objc %}
vi sites-available/mobile.conf  
########################################################  
\# Server Directives #  
########################################################  
server {  
listen 80;  
server_name m.example.com;

########################################################  
\# Location Directives #  
########################################################  
location / {  
uwsgi_pass unix://var/run/mobile.sock;  
include uwsgi_params;  
}

location /static/ {  
alias /var/www/mobile/MobileTicketSales/static/;  
}

location /s/ {  
alias /var/www/mobile/MobileTicketSales/static/;  
}

location /media/ {  
alias /var/www/mobile/MobileTicketSales/media/;  
}  
}  
For additional security add the following to your site configuration file. This should be added above the &#8220;Location&#8221; Directive  
########################################################  
\# Security Directives #  
########################################################  
\# Only requests to m.example.com are allowed #  
if ($host !~ ^(example.com|m.example.com)$ )  
{  
return 444;  
}

#Only allow these request methods  
if ($request_method !~ ^(GET|HEAD|POST)$ )  
{  
return 444;  
}

#Block download agents  
if ($http\_user\_agent ~* LWP::Simple|BBBike|wget|Baiduspider|Jullo)  
{  
return 403;  
}

#Block some robots  
if ($http\_user\_agent ~* msnbot|scrapbot)  
{  
return 403;  
}

#Deny certain Referers  
if ( $http_referer ~* (babes|forsale|girl|jewelry|love|nudit|organic|poker|porn|sex|teen) )  
{  
return 403;  
}  
{% endcodeblock %}

Enable your new Site  
{% codeblock lang:objc %}
cd sites-enabled  
ln -s ../sites-available/mobile.conf .  
{% endcodeblock %}

Modify Nginx init with the new installation PATH  
{% codeblock lang:objc %}
vi /etc/init.d/nginx  
\# Add the following to PATH  
:/usr/local/nginx  
{% endcodeblock %}

Update Site permissions  
{% codeblock lang:objc %}
chown -R www-data:www-data /var/www/mobile  
{% endcodeblock %}

Start/restart your Nginx Service  
{% codeblock lang:objc %}
service nginx restart  
{% endcodeblock %}
