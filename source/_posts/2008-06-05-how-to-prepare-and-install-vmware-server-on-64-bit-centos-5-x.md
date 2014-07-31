---
title: How to prepare and install VMware Server on 64-bit CentOS 5.x
author: Mike Moghadas
layout: post
categories:
  - Linux
  - VMware
tags:
  - CentOS
---
STEP 1.  
\# cat /etc/redhat-release  
CentOS release 5 (Final)

STEP 2.  
\# uname -a  
Linux server.example.com 2.6.18-53.1.21.el5 #1 SMP Tue May 20 09:35:07 EDT 2008 x86\_64 x86\_64 x86_64 GNU/Linux

<!--more-->

STEP 3.  
make sure you have the following development packages: glibc gcc gcc-c++  
Install them by running the following:  
\# yum install glibc gcc gcc-c++

STEP 4.  
Make sure you have kernel, kernel-devel and kernel-headers. All need to be the same version! Pretty much you need to have kernel-devel and kernel-headers packages installed for your kernel listed in step 2.  
\# rpm -qa | grep kernel  
kernel-devel-2.6.18-53.1.21.el5  
kernel-2.6.18-53.1.21.el5  
kernel-headers-2.6.18-53.1.21.el5

If you don&#8217;t have all of above install them by running the following: (note that this will install a most recent packages including the kernel. If you can&#8217;t update the kernel for some reason, install the header and devel packages seperately and can be downloade from centos.org)  
\# yum install kernel kernel-devel kernel-headers

If you install a new kernel, you will need to reboot!

STEP 5.  
Install other required packages: libXtst-devel, libXrender-devel and xinetd  
\# yum install libXtst-devel libXrender-devel xinetd

Set xinetd to start at boot  
\# chkconfig xinetd on

Start xinetd service  
\# service xinetd start

STEP 6.  
Download and install VMware Server  
\# mkdir installs  
\# wget http://vmware-server-location-site/VMware-server-1.0.5-80187.i386.rpm (or your version)  
\# rpm -ivh VMware-server-1.0.5-80187.i386.rpm

Run the configuration tool and setup based on your preference  
\# vmware-config.pl

STEP 7.  
You should now be able to connect to the vm server using VMware Server Console from anywhere and on the port you specified during configuration.
