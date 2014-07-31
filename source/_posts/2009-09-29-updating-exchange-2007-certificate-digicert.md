---
title: Updating Exchange 2007 Certificate (DigiCert)
author: Mike Moghadas
layout: post
categories:
  - Exchange
tags:
  - certificate
  - Exchange 2010
  - ssl
---
**Creating CSR (Request)**

Visit DigiCert&#8217;s Exchange 2007 CSR Tool @ https://www.digicert.com/easy-csr/exchange2007.htm  
Fill the form and generate the &#8220;command&#8221; to create the CSR on Exchange 2007. The command should look like this 

<!--more-->

{% codeblock lang:objc %}
New-ExchangeCertificate -GenerateRequest -Path c:mailserver\_subdomain\_example_com.csr -KeySize 2048 -SubjectName &#8220;c=US, s=California, l=Campbell, o=Company LLC, ou=IT, cn=mailserver.subdomain.example.com&#8221; -DomainName mailserver.subdomain.example.com, autodiscover.example.com, autodiscover.example.com, webmail.example.com, mail.example.com, mailserver -PrivateKeyExportable $True  
{% endcodeblock %}

Copy the generated command into &#8220;Exchange Management Shell&#8221;. The file &#8220;c:mailserver\_subdomain\_example_com.csr&#8221; is created and needs to be sent to DigiCert

**Importing CER (Certificate)**

Once you have the signed certificate&#8230;

1. Copy the acquired certificate to drive accessible to Exchange. i.e. C:mailserver\_subdomain\_example_com.cer

2. Run the following import command and enable required services (IIS and SMTP)  
{% codeblock lang:objc %}
Import-ExchangeCertificate -Path C:mailserver\_subdomain\_example_com.cer | Enable-ExchangeCertificate -Services &#8220;SMTP, IIS&#8221;  
{% endcodeblock %}

3. Confirm the services  
{% codeblock lang:objc %}
Get-ExchangeCertificate -DomainName mailserver.subdomain.example.com  
Thumbprint Services Subject  
&#8212;&#8212;&#8212;- &#8212;&#8212;&#8211; &#8212;&#8212;-  
&#8230;.  
1113D77630A34FDEA3D465779897431694204423 IP.WS CN=mailserver.subdomain.example.com, OU=IT, O=Company LLC, L=Camp&#8230;  
&#8230;.  
{% endcodeblock %}

Enable services if missing  
{% codeblock lang:objc %}
4. Enable-ExchangeCertificate -Services SMTP, IIS -thumbprint 1113D77630A34FDEA3D465779897431694204423  
{% endcodeblock %}
