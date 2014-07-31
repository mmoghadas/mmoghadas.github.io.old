---
title: Upgrading Ubuntu 9.04 to 10.04 LTS
author: Mike Moghadas
layout: post
categories:
  - Linux
tags:
  - Linux
  - Ubuntu
  - Upgrade
---
NOTE: Upgrading directly from 9.04 to 10.04 LTS is not possible. The proper upgrade path is 9.04 > 9.10 > 10.04. 

**Upgrade 9.04 to 9.10 first**  
1. Download and mount ubuntu-9.10-alternate-i386.iso  
2. Mount your iso  
{% codeblock lang:objc %}
mount -o loop ~/ubuntu-9.10-alternate-i386.iso /media/cdrom  
mount -o loop /dev/cdrom /media/cdrom (if your burned your iso)  
{% endcodeblock %}

<!--more-->

3. After mounting the iso, you can simply run this utility to upgrade to 9.10  
{% codeblock lang:objc %}
/media/cdrom/cdromupgrade  
{% endcodeblock %}  
4. Select the following options during upgrade process  
- Start additional SSH daemon = YES  
- Include latest updates from the Internet = NO  
- The rest is all YES  
5. Reboot

**Upgrade 9.10 to 10.04LTS**  
6. Install the required package  
{% codeblock lang:objc %}
apt-get install update-manager-core  
{% endcodeblock %}  
7. Modify the release-upgrades  
{% codeblock lang:objc %}
vi /etc/update-manager/release-upgrades  
set Prompt=normal  
do-release-upgrade  
{% endcodeblock %}  
8. Reboot  
9. To make sure everything is up-to-date  
{% codeblock lang:objc %}
apt-get update  
apt-get upgrade  
apt-get -f install  
{% endcodeblock %}
