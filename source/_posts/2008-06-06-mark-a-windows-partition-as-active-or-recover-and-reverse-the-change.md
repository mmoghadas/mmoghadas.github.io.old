---
title: Mark a Windows partition as active or Recover and Reverse the change
author: Mike Moghadas
layout: post
categories:
  - Windows
tags:
  - Partition
---
Have you accidently marked a partition as active in a Windows 2003 Server? It’s easy to reverse what you’ve done: USE DISKPART TO FIX IT.

<!--more-->

1. Open Command Prompt  
2. Type:  
\# DISKPART

3. List your disks  
\# LIST DISK

4. Select the disk your unwanted “Active” partition is on  
\# SELECT DISK 

5. List your disk’s partitions  
\# LIST PARTITION

6. Type:  
\# INACTIVE
