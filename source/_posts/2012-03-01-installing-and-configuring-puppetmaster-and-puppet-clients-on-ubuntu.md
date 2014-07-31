---
title: Installing and configuring Puppetmaster and Puppet Clients (On Ubuntu)
author: Mike Moghadas
layout: post
categories:
  - Linux
  - Puppet
tags:
  - master
  - puppet
---
Puppet is a great Client/Server automation tool for your *NIX environment. The tool can be used to automate:

Installations  
Updates  
Deployment  
Service manipulation  
etc&#8230;

NOTE: Typically, the client(puppet/node) retrieves updates from the server (Puppetmaster). However you can enable push from the server to client as well. See &#8220;Client Configuration&#8221;.

Follow the below steps to get a basic configuration going. 

<!--more-->

**Installation: Server**  
{% codeblock lang:objc %}
apt-get install puppetmaster  
{% endcodeblock %}

**Installation: Client**  
{% codeblock lang:objc %}
apt-get install puppet  
{% endcodeblock %}

**Note: ***Although not required, you may want to make sure you have the same version of Puppet and Puppetmaster. I have Puppet 2.6.3 and Puppetmaster 2.7.1 running sucessfully, but have had issues with previous versions and installations.* 

If you like to install the same exact version of client/server you have some options:  
1. Install via Ruby  
{% codeblock lang:objc %}
apt-get install build-essential zlib1g zlib1g-dev libxml2 libxml2-dev curl wget openssl libssl-dev libopenssl-ruby  
mkdir /root/installs  
cd /root/installs  
wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.2-p180.tar.gz  
tar zxvf ruby-1.9.2-p180.tar.gz  
cd ruby-1.9.2-p180  
./configure  
make  
make install  
gem update &#8211;system

gem install facter  
gem install puppet  
{% endcodeblock %}

2. Install from source (Assuming you have ruby installed)  
{% codeblock lang:objc %}
wget http://puppetlabs.com/downloads/facter/facter-latest.tgz  
gzip -d -c facter-latest.tgz | tar xf -  
cd facter-*  
ruby install.rb

wget http://puppetlabs.com/downloads/puppet/puppet-latest.tgz  
gzip -d -c puppet-latest.tgz | tar xf -  
ruby install.rb  
{% endcodeblock %}

3. Another way to install  
{% codeblock lang:objc %}
apt-get install ruby rubygems libopenssl-ruby  
export RUBYOPT=rubygems  
gem install facter puppet  
{% endcodeblock %}

4. Install form 3rd Party repository  
{% codeblock lang:objc %}
apt-get install python-software-properties  
add-apt-repository ppa:mathiaz/puppet-backports  
apt-get update  
apt-get install puppetmaster  
-OR-  
apt-get install puppet  
{% endcodeblock %}

**Configuration: Server**  
Edit the puppet.conf file and add the following lines  
{% codeblock lang:objc %}
vi /etc/puppet/puppet.conf  
[agent]  
classfile = $vardir/classes.txt  
localconfig = $vardir/localconfig  
report = true

[master]  
autosign = true  
{% endcodeblock %}

Edit the fileserver.conf file and add the following lines  
{% codeblock lang:objc %}
vi /etc/puppet/fileserver.conf

[files]  
path /etc/puppet/files  
allow 10.176.0.0/16

[plugins]  
allow 10.176.0.0/16

[modules]  
allow 10.176.0.0/16  
{% endcodeblock %}

Create the files directory specified above (required to start the puppetmaster service)  
{% codeblock lang:objc %}
mkdir /etc/puppet/files  
{% endcodeblock %}

Configure your nodes (client) below. The templates will apply to these nodes only.  
{% codeblock lang:objc %}
vi /etc/puppet/manifests/nodes.pp

node &#8216;basenode&#8217; {  
include baseclass  
}

node &#8216;mobile101.example.com&#8217; inherits basenode {  
}

node &#8216;mobile102.example.com&#8217; inherits basenode {  
}  
{% endcodeblock %}

Configure your templates. Here you&#8217;re listing the modules will effect your clients.  
{% codeblock lang:objc %}
vi /etc/puppet/manifests/templates.pp

node default {  
include baseclass  
}

class baseclass {  
include mobile::mobile_content  
include mobile::mobile_settings  
include mobile::mobile\_wsgi\_app  
include mobile::mobile_django  
}  
{% endcodeblock %}

Edit the site.pp and import your nodes and templates  
{% codeblock lang:objc %}
vi /etc/puppet/manifests/site.pp

import &#8220;nodes&#8221;  
import &#8220;templates&#8221;

filebucket { main: server => puppet }  
{% endcodeblock %}

Restart puppetmaster to enable the above changes  
{% codeblock lang:objc %}
/etc/init.d/puppetmaster restart  
{% endcodeblock %}

**Configuration: Client**  
There are only a few steps to get the client up and running. First up is the puppet main configuration file. Edit the file below and make changes.  
{% codeblock lang:objc %}
vi /etc/puppet/puppet.conf  
\# VERY IMPORTANT&#8230; THE &#8220;SERVER&#8221; VARIABLE MUST BE USING THE HOSTNAME OF THE PUPPETMASTER..NOT AN ALIAS!!

[main]  
server = puppetmaster.example.com

#[puppetmasterd]  
#templatedir=/var/lib/puppet/templates

[agent]  
report = true  
listen = true  
runinterval = 3600 #1 hour  
{% endcodeblock %}

Configure puppet to automatically at boot  
{% codeblock lang:objc %}
vi /etc/default/puppet  
START=yes  
{% endcodeblock %}

Create the following required file  
{% codeblock lang:objc %}
touch /etc/puppet/namespaceauth.conf  
{% endcodeblock %}

Created the following file if you like to push content manually from the server.  
{% codeblock lang:objc %}
vi /etc/puppet/auth.conf  
path /run  
allow puppetmaster.example.com  
{% endcodeblock %}

In this example, I&#8217;m using upstart to recycle a service. You&#8217;ll to copy &#8220;upstart.rb&#8221; from the server to client.  
{% codeblock lang:objc %}
cd /usr/lib/ruby/1.8/puppet/provider/service/  
scp username@puppetmaster.example.com:/usr/lib/ruby/1.8/puppet/provider/service/upstart.rb .  
{% endcodeblock %}

**Configuration: Certificate**  
STEP1: On the Client:  
{% codeblock lang:objc %}
puppetd &#8211;server puppetmaster.example.com &#8211;waitforcert 60  
{% endcodeblock %}

STEP2: On the Server:  
{% codeblock lang:objc %}
puppetca -la  
puppetca &#8211;sign mobile101.example.com  
puppetca &#8211;sign mobile102.example.com  
{% endcodeblock %}

STEP3: On the Client:  
{% codeblock lang:objc %}
pkill puppet  
/etc/init.d/puppet restart  
{% endcodeblock %}

STEP4: Only if necessary: Troubleshooting Certificate Issues:  
{% codeblock lang:objc %}
#On the server  
puppetca -r mobile101.example.com  
puppetca -c mobile101.example.com  
/etc/init.d/puppetmaster restart

#On the client  
pkill puppetd  
rm /var/lib/puppet/ssl/ -rf  
{% endcodeblock %}  
Repeat the certificate creation process

**Configuration: Modules**  
At this point there are no configuration left on the client. As long as the certificate was configured properly all changes make beyond this point are all on the server. Lets configure a mobile module we included in the configuration above. Lets start off with the required directory structure.  
{% codeblock lang:objc %}
mkdir /etc/puppet/files  
mkdir /etc/puppet/modules  
mkdir /etc/puppet/modules/mobile/  
mkdir /etc/puppet/modules/mobile/files  
mkdir /etc/puppet/modules/mobile/manifests  
{% endcodeblock %}

The files directory structure is listed below. In this case we&#8217;re assuming you&#8217;re going to be deploying a website named &#8220;mobile&#8221; with it&#8217;s root/content of &#8220;var/www/mobile&#8221;. You can touch these files for this example.  
{% codeblock lang:objc %}
ls -m /etc/puppet/modules/mobile/*  
/etc/puppet/modules/mobile/files:  
mobile

/etc/puppet/modules/mobile/manifests:  
config.pp, init.pp, service.pp  
{% endcodeblock %}

Modify the init.pp file  
{% codeblock lang:objc %}
class mobile {  
include mobile::config, mobile::service  
}  
{% endcodeblock %}

Modify the config.pp file  
{% codeblock lang:objc %}
class mobile::config {  
file { &#8220;/var/www/mobile&#8221;:  
ensure => present,  
source => &#8220;puppet:///modules/mobile/mobile&#8221;,  
recurse => true,  
group => &#8220;www-data&#8221;,  
owner => &#8220;www-data&#8221;,  
mode => &#8220;0644&#8243;  
}

file { &#8220;/var/www/mobile/MobileTicketSales/settings.py&#8221;:  
ensure => present,  
source => &#8220;puppet:///modules/mobile/settings.py&#8221;,  
recurse => true,  
group => &#8220;www-data&#8221;,  
owner => &#8220;www-data&#8221;,  
mode => &#8220;0644&#8243;  
}

file { &#8220;/var/www/mobile/MobileTicketSales/wsgi_app.py&#8221;:  
ensure => present,  
source => &#8220;puppet:///modules/mobile/wsgi_app.py&#8221;,  
recurse => true,  
group => &#8220;www-data&#8221;,  
owner => &#8220;www-data&#8221;,  
mode => &#8220;0644&#8243;  
}

file { &#8220;/var/www/mobile/MobileTicketSales/django.wsgi&#8221;:  
ensure => present,  
source => &#8220;puppet:///modules/mobile/django.wsgi&#8221;,  
recurse => true,  
group => &#8220;www-data&#8221;,  
owner => &#8220;www-data&#8221;,  
mode => &#8220;0644&#8243;  
}

}  
{% endcodeblock %}

Modify the service.pp file  
{% codeblock lang:objc %}
class mobile::service {  
service { &#8220;mobile&#8221;:  
provider => &#8220;upstart&#8221;,  
ensure => running,  
hasstatus => true,  
hasrestart => true,  
enable => true,  
subscribe => File["/var/www/mobile/MobileTicketSales/settings.py"],  
#require => Class["mobile::config"],  
}  
}  
{% endcodeblock %}

That&#8217;s all she wrote. Your clients should retrive any changes hourly (based on configuration above). If you like to push content manually from the server:  
{% codeblock lang:objc %}
puppetrun &#8211;host mobile101.example.com  
{% endcodeblock %}
