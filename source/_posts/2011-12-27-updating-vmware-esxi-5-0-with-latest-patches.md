---
title: Updating VMware ESXI 5.0 with latest patches
author: Mike Moghadas
layout: post
categories:
  - Linux
  - VMware
tags:
  - ESXi
  - patch
  - vmware
---
VMware was nice enough to remove the pretty &#8220;GUI&#8221; Update Utility after ESXi version 4.1.  So now you&#8217;ll need to do bunch of commands to get your VMware Host Updated.  Here&#8217;s how:

**What you&#8217;ll need**  
1. Install VMware CLI on your machine. Once this tool is installed the required utilities are located in &#8220;**C:Program Files (x86)VMwareVMware vSphere CLIbin**&#8220;.  Add this location to your PATH to make life easier. (The below instructions assume you&#8217;ve done this)

<!--more-->

2. ESXi Patch you want to apply downloaded from VMware

3. SSH Enabled on the ESXi Host(s)  
**How to Patch things up **

1. Copy your patch to the ESXi Host.  You can use scp/ssh to do this  
{% codeblock lang:objc %} scp ESXi500-201111001.zip root@10.10.10.206:/vmfs/volumes/datastore1/ {% endcodeblock %}  
2. Force your EXSi to enter Maintenance Mode  
{% codeblock lang:objc %} vicfg-hostops.pl &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** &#8211;operation enter {% endcodeblock %}  
3. Verify Maintenance Mode  
{% codeblock lang:objc %} vicfg-hostops.pl &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** &#8211;operation info {% endcodeblock %}  
&#8211;OR&#8211;  
{% codeblock lang:objc %} vicfg-hostops.pl &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** &#8211;operation info | grep &#8220;In Maintenance Mode&#8221; {% endcodeblock %}  
4. Update VM Host with your new patches  
{% codeblock lang:objc %} esxcli &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** software vib update &#8211;depot=/vmfs/volumes/datastore1/ESXi500-201111001.zip {% endcodeblock %}  
5. Reboot VM Host  
{% codeblock lang:objc %} vicfg-hostops.pl &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** &#8211;operation reboot {% endcodeblock %}  
6. Exit Maintenance Mode  
{% codeblock lang:objc %} vicfg-hostops.pl &#8211;server=10.10.10.206 &#8211;username=root &#8211;password=\***\*** &#8211;operation exit {% endcodeblock %}
