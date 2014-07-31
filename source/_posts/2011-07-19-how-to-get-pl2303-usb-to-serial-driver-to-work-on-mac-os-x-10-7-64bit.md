---
title: How to get PL2303 USB to Serial Driver to work on Mac OS X 10.7 (64bit)
author: Mike Moghadas
layout: post
categories:
  - OSX
tags:
  - 10.7
  - 64bit
  - PL2303
  - USB
---
**What you need**  
File: osx-pl2303.kext.zip  
Location: <a href="http://sourceforge.net/tracker/index.php?func=detail&#038;aid=2952982&#038;group_id=157692&#038;atid=804837" target="_blank">http://sourceforge.net/tracker/index.php?func=detail&aid=2952982&group_id=157692&atid=804837</a>

**How to get it working**  
1. Download the above driver for your 64bit OSX  
2. Unpack the driver  
3. Copy osx-pl2303.kext to /System/Library/Extensions  
{% codeblock lang:objc %}
sudo cp -r ~/osx-pl2303.kext /System/Library/Extensions/  
{% endcodeblock %}

<!--more-->

4. Load the driver  
{% codeblock lang:objc %}
sudo kextload /System/Library/Extensions/osx-pl2303.kext  
{% endcodeblock %}

5. Plug your USB-to-Serial in  
6. Test  
{% codeblock lang:objc %}
ls -l /dev/cu.PL2303-*  
or  
ls -l /dev/tty.PL2303-*  
{% endcodeblock %}  
7. If you see a device with above name, you have installed your driver successfully
