---
title: How to monitor MySQL Replication
author: Mike Moghadas
layout: post
categories:
  - Linux
  - MySQL
tags:
  - MySQL
  - Replication
  - SNMP
  - SNMPTrap
---
I have spent some time looking for a solution to monitor MySQL Replication with no lock. So I decided to figure something out myself rather than wasting more time or end up paying Oracle. Turns out the setup is actually very simple and can be easily setup with few lines of shell script and a monitoring tool that can produce alerts and graphs via SNMP Traps (PRTG in my case).

**What do you need?**  
*Note: Make sure to update the below addresses*  
MySQL Master: 192.168.1.101  
MySQL Slave: 192.168.1.102  
SNMP Tools (snmptrap specifically)  
Monitoring Server (I use PRTG): 192.168.1.116  
Your shell script  
A Replication client user must be created on both Master and Slave servers.

<!--more-->

**Where do you configure things?**  
You can run the script/cronjob from either Master or the Slave. I configured the job on the Slave server keep the load off the Master.

**Step1:** Configure Replication. I’m not going to cover replication here. Plenty of articles available on how to configure replication

**Step2:** Create the Replication Client user on Master and Slave. Run the following command on both servers. Make sure to grant REPLICATION CLIENT privilege only.

*Note: replace the “\***\***\*****” with your password for the new account.*  
{% codeblock lang:objc %}
mysql> GRANT REPLICATION CLIENT ON \*.\* TO &#8216;rep-client&#8217;@&#8217;192.168.1.101&#8242; IDENTIFIED BY &#8216;\***\***\*****&#8217;;  
{% endcodeblock %}

**Step3:** Test your newly created account’s permissions. Logon to the Slave server and run the following commands Master and the Slave:  
{% codeblock lang:objc %}
mysql -urep-client -p\***\***\***** -h192.168.1.101 -e &#8220;show master statusG;&#8221;  
mysql -urep-client -p\***\***\***** -h192.168.1.102 -e &#8220;show slave statusG;&#8221;  
{% endcodeblock %}

**Step4:** Create the following script  
{% codeblock lang:objc %}
vi /root/scripts/MySQL\_Replication\_Check  
#!/bin/bash

#MASTER Variables  
M_Position=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.101 -e &#8220;show master statusG;&#8221; | grep Position | sed -e &#8216;s/[^:]\*://&#8217;\`  
M_File=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.101 -e &#8220;show master statusG;&#8221; | grep File | sed -e &#8216;s/[^:]\*://&#8217;\`

#SLAVE Variables  
S\_Exec\_Master\_Log\_Pos=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.102 -e &#8220;show slave statusG;&#8221; | grep Exec\_Master\_Log_Pos | sed -e &#8216;s/[^:]\*://&#8217;\`  
S\_Master\_Log\_File=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.102 -e &#8220;show slave statusG;&#8221; | grep Master\_Log\_File | sed -e &#8216;/Relay\_Master\_Log\_File/d&#8217; | sed -e &#8216;s/[^:]\*://&#8217;\`  
S\_Slave\_IO\_State=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.102 -e &#8220;show slave statusG;&#8221; | grep Slave\_IO_State | sed -e &#8216;s/[^:]\*://&#8217;\`  
S\_Seconds\_Behind\_Master=\`mysql -urep-client -\***\***\***\***\****\* -h192.168.1.102 -e &#8220;show slave statusG;&#8221; | grep Seconds\_Behind_Master | sed -e &#8216;s/[^:]\*://&#8217;\`  
S\_Slave\_IO\_StateString=\`echo $S\_Slave\_IO\_State\`  
S\_Slave\_IO_StateOK=&#8221;Waiting for master to send event&#8221;

#Replication Overall Status. Send SNMPTrap to Monitoring  
echo &#8220;&#8221;  
if [ "$S\_Slave\_IO\_StateString" == "$S\_Slave\_IO\_StateOK" ] && [ $M\_Position -eq $S\_Exec\_Master\_Log\_Pos ] && [ $M\_File == $S\_Master\_Log\_File ] && [ $S\_Seconds\_Behind\_Master -lt 5 ]  
then  
echo &#8220;Replication is running fine&#8230;.&#8221;  
snmptrap -v 1 -c ROpublic 192.168.1.99 &#8221; &#8221; &#8221; &#8221; &#8221; 1.11.12.13.14.15111 s &#8220;Replication OK&#8221;  
else  
echo &#8220;Replication BROKEN&#8221;  
snmptrap -v 1 -c ROpublic 192.168.1.99 &#8221; &#8221; &#8221; &#8221; &#8221; 1.11.12.13.14.15111 s &#8220;Replication Broken&#8221;  
fi

#Replication State Status  
echo &#8220;&#8221;  
echo &#8220;Slave IO State: &#8221; $S\_Slave\_IO_StateString  
if [ "$S\_Slave\_IO\_StateString" == "$S\_Slave\_IO\_StateOK" ]  
then  
echo &#8220;Replication OK&#8221;  
else  
echo &#8220;Replication STOPPED!&#8221;  
fi

#Replication Log Files Status  
echo &#8220;&#8221;  
echo &#8220;Master Log File: &#8221; $M_File  
echo &#8220;Slave Log File: &#8221; $S\_Master\_Log_File  
if [ $M\_File == $S\_Master\_Log\_File ]  
then  
echo &#8220;Log Files: OK&#8221;  
else  
echo &#8220;Log Files DO NOT Match!&#8221;  
fi

#Replication Log Position Status  
echo &#8220;&#8221;  
echo &#8220;Master Position: &#8221; $M_Position  
echo &#8220;Slave Position: &#8221; $S\_Exec\_Master\_Log\_Pos  
if [ $M\_Position -eq $S\_Exec\_Master\_Log_Pos ]  
then  
echo &#8220;Position: OK&#8221;  
else  
echo &#8220;Slave is Behind!&#8221;  
fi

#Slave Seconds Behind Master  
echo &#8220;&#8221;  
echo &#8220;Slave Seconds Behind Master: &#8221; $S\_Seconds\_Behind_Master  
if [ $S\_Seconds\_Behind_Master -lt 5 ]  
then  
echo &#8220;Acceptable: YES&#8221;  
else  
echo &#8220;Acceptable: NO, Slave is Way Behind!&#8221;  
fi  
{% endcodeblock %}

**Step5:** Create a cronjob to run your new script every 60 seconds. You can adjust this if you like.. All depends on how busy your database is.  
{% codeblock lang:objc %}
crontab -e  
\# MySQL Replication Check  
\* \* \* \* * /root/scripts/MySQL\_Replication\_Check  
{% endcodeblock %}

**Step6:** Configure your monitoring to receive SNMPTraps from your slave (or where you are running your cronjob). The following is meant to be used with PRTG. You can configure other monitoring tools to received SNMP Traps as well.  
- Logon to the Web Interface  
- Devices  
- Select your Probe  
- Add Sensor  
- Select or search for &#8220;SNMP Trap Receiver&#8221; and complete the following:  
OID value: 1.11.12.13.14.15111  
Sender: 192.168.1.102  
Set sensor to &#8216;warning&#8217;/Message must include: Replication OK  
Set sensor to &#8216;warning&#8217;/Message must not include: Replication Broken  
Scanning Interval: 60 seconds

**Step7:** Make sure you have SNMP Tools installed.  
On Redhat:  
{% codeblock lang:objc %}
yum install net-snmp-utils  
{% endcodeblock %}
