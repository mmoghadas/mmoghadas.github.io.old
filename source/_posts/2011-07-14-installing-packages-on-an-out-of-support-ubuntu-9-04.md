---
title: Installing packages on an out-of-support Ubuntu 9.04
author: Mike Moghadas
layout: post
categories:
  - Linux
tags:
  - Repository
  - Ubuntu
---
By now your 9.04 (Jaunty) is out of support and at some point you might want to install a package from Ubuntu repositories. You will need need to tweak a few things before you can install or upgrade your packages. Here&#8217;s an easy way to this:

**Edit and update where Ubuntu gets it&#8217;s packages from**  
{% codeblock lang:objc %}
sudo vi /etc/apt/sources.list  
:%s/us.archive.ubuntu/old-releases.ubuntu/g  
:%s/security.ubuntu/old-releases.ubuntu/g  
{% endcodeblock %}

**Update and resynchronize the package indexes**  
{% codeblock lang:objc %}
sudo apt-get update  
{% endcodeblock %}

Now you should be able to install or upgrade packages
