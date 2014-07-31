---
title: Enabling SMTP Relay on Exchange 2007
author: Mike Moghadas
layout: post
categories:
  - Exchange
tags:
  - Exchange 2007
  - SMTP Relay
---
To Enable Relaying on port 486 for Local Network (10.10.10.0/24) on Exchange Serever.

Exchange Management Console > Server Configuration > Hub Transport  
Select MailServer Hub Transport  
Select Internal Relay MailServer Receive Connector > Properties > Network  
Add 10.10.10.0/24 to Receive mail form remote servers&#8230;
