---
title: Configure Cisco WebVPN to use a Signed Certificate
author: Mike Moghadas
layout: post
categories:
  - Cisco
  - Networking
tags:
  - 1811
  - certificate
  - cisco
  - signed
  - ssl
---
If you like to install a Signed Certificate on your cisco Router&#8230;

1.  Create a new CSR on Test IIS Server
2.  Submit CSR to CA
3.  Once Signed/Generated, download the newly generated PKCS12 certificate and import to IIS
4.  Export the new certificate via MMC  
    *      &#8211; Export *.example.com certificate from “Personal Certificate” with key*  
    *      &#8211; Include all certificates in the path (CA, Intermediate)*
5.  Place the exported .pfx file onto a tftp server
6.  On the Cisco Router:

<!--more-->

{% codeblock lang:objc %}
crypto pki import ExampleWildcard-Crt pkcs12 tftp://10.10.10.65/certificate_file  
!Yes to all  
{% endcodeblock %}

Updated the WebVPN configuration to use the new certificate:  
{% codeblock lang:objc %}
webvpn gateway VTX-Admins  
ssl trustpoint ExampleWildcard-Crt  
{% endcodeblock %}
