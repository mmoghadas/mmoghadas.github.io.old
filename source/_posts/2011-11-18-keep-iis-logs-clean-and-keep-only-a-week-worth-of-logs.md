---
title: Keep IIS Logs Clean and keep only a week worth of logs
author: Mike Moghadas
layout: post
categories:
  - Uncategorized
tags:
  - IIS
  - LOGs
  - Rotate
  - Windows
---
Webserver logs can add up and eventually fill up your entire disk if not mananged properly. Windows and IIS don&#8217;t exactly give you flexible options to manage logs either. So what to do? Here&#8217;s simple solution. We&#8217;re going to create couple of scripts and kick off a Task to purge old logs.

Lets create a folder to store the scripts  
{% codeblock lang:objc %}
mkdir C:scripts  
{% endcodeblock %}

<!--more-->

Create a batch file to kick off a powershell script  
{% codeblock lang:objc %}
notepad C:scriptsDelete-Old-IIS-Logs.bat #Add the below line to the .bat file  
powershell.exe C:scriptsDelete-Old-IIS-Logs.ps1  
{% endcodeblock %}

Lets create a powershell script that will clean up the logs and leave 7 days worth  
{% codeblock lang:objc %}
notepad C:scriptsDelete-Old-IIS-Logs.ps1 #Add the below lines to the .ps1 file  
$Path = &#8220;C:inetpublogsLogFilesW3SVC1&#8243;  
$LogsToKeep = &#8220;7&#8243;  
$CurrentDate = Get-Date  
$DatetoDelete = $CurrentDate.AddDays(-$LogsToKeep)

Get-ChildItem $Path -recurse | Where-Object { $_.LastWriteTime -lt $DatetoDelete } | Remove-Item  
{% endcodeblock %}

Create a new Task via Task Scheduler and Check the following settings:  
- Run whether user is logged on or not  
- Run with highest privileges  
- Action should be: Start a Program and pointing to &#8220;:scriptsDelete-Old-IIS-Logs.bat&#8221;
