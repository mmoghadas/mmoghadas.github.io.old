---
title: Fresh Installation of Bugzilla on Ubuntu
author: Mike Moghadas
layout: post
categories:
  - Linux
  - Web Server
tags:
  - Apache2
  - Bugzilla
---
**Install required packages**  
{% codeblock lang:objc %}
apt-get install perl mysql-server mysql-client mysql-admin apache2  
apt-get install build-essential libcgi-pm-perl libdigest-sha1-perl timedate libdatetime-event-sunrise-perl libdbi-perl libhtml-template-perl libemail-send-perl libemail-mime-perl libemail-mime-encodings-perl libemail-mime-modifier-perl liburi-perl libdbd-mysql-perl libtoolkit-perl libtemplate-perl libgd-gd2-perl libchart-perl libtemplate-plugin-gd-perl libxml-twig-perl libmime-tools-perl libwww-perl perlmagick libnet-ldap-perl libauthen-sasl-perl libauthen-simple-radius-perl libsoap-lite-perl libhtml-parser-perl libhtml-scrubber-perl libtheschwartz-perl libapache2-mod-perl2 libfile-slurp-perl libfile-flock-perl libyaml-perl  
{% endcodeblock %}

<!--more-->

**Configure Database**  
{% codeblock lang:objc %}
$ mysql -u root -p  
mysql> create database bugzilla;  
mysql> grant all privileges on bugzilla.* to bugzilla@localhost identified by &#8216;bugzilla&#8217;;  
{% endcodeblock %}

**Download and untar bugzilla **  
{% codeblock lang:objc %}
$ tar -zxvf bugzilla-3.x  
$ mv bugzilla-3.x /usr/local/bugzilla-3.x  
$ ln -s /usr/local/bugzilla3.x /var/www/bugzilla  
$ cd /usr/local/bugzilla-3.x  
$ ./checksetup.pl &#8211;check-modules  
{% endcodeblock %}

**Most likely the installer will complain about missing modules. Here&#8217;s how you can install them:**  
{% codeblock lang:objc %}
/usr/bin/perl install-module.pl Daemon::Generic  
/usr/bin/perl install-module.pl Email::Reply  
/usr/bin/perl install-module.pl PatchReader  
/usr/bin/perl install-module.pl Email::MIME::Attachment::Stripper  
/usr/bin/perl install-module.pl Template  
{% endcodeblock %}

**Recheck for missing modules and if successful, continue with the install**  
{% codeblock lang:objc %}
$ ./checksetup.pl &#8211;check-modules  
$ ./checksetup.pl  
{% endcodeblock %}

**Edit bugzilla&#8217;s configuration file and enter your database information**  
{% codeblock lang:objc %}
$ vim localconfig  
Edit following  
$webservergroup = &#8216;www-data&#8217;;  
$db_name = &#8216;bugzilla&#8217;;  
$db_user = &#8216;bugzilla&#8217;;  
$db_pass = &#8216;bugzilla&#8217;;  
{% endcodeblock %}

**Modify the database to support larger attachement sizes**  
{% codeblock lang:objc %}
$ mysql -u root -p  
mysql> use bugzilla;  
mysql> alter table attachments avg\_row\_length=1000000, max_rows=20000;  
{% endcodeblock %}

**Configure Apache2 and your VirtualHost**  
{% codeblock lang:objc %}
$ ./checksetup.pl  
$ vim /etc/apache2/sites-available/bugzilla 

<VirtualHost *:80>  
ServerName bugzilla.test.example.com  
ServerAlias bugs bugs.test bugs.test.example.com bugzilla bugzilla.fs  
ServerAdmin root@example.com  
DocumentRoot /var/www/bugzilla  
ErrorLog &#8220;|/usr/sbin/rotatelogs /var/log/apache2/bugzilla.%Y-%m-%d.error 86400&#8243;  
CustomLog &#8220;|/usr/sbin/rotatelogs /var/log/apache2/bugzilla.%Y-%m-%d.log 86400&#8243; common  
</VirtualHost>  
Alias /bugzilla/ /var/www/bugzilla/  
<Directory /var/www/bugzilla>  
AddHandler cgi-script .cgi .pl  
Options +Indexes +ExecCGI +FollowSymLinks  
DirectoryIndex index.cgi  
AllowOverride Limit  
</Directory>  
{% endcodeblock %}

**Create a symbolic link to enable bugzilla&#8217;s VirtualHost **  
{% codeblock lang:objc %}
ln -s /etc/apache2/sites-available/bugzilla /etc/apache2/sites-enables/bugzilla  
{% endcodeblock %}

**Start/Restart Apache**  
{% codeblock lang:objc %}
/etc/initd./apache2 restart  
{% endcodeblock %}
