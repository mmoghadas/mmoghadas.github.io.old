---
title: F5 Certificate Import Process
author: Mike Moghadas
layout: post
categories:
  - Networking
tags:
  - certificate
  - F5
  - ssl
---
Note: This is assuming you currently have a valid certificate installed on some IIS Server

1. Export example.com certificate via IIS to PFX. Make sure to include the certificate and the private key. If exporting a bundled certificate, you will need to copy/paste the example.com certificate below only

2. Convert PFX to PEM 

<!--more-->

{% codeblock lang:objc %}
openssl pkcs12 -in example\_all.pfx -out example\_all_pem.cer –nodes  
{% endcodeblock %}

3. Import Cert via PEM to f5  
Local Traffic > SSL Certificates > Import  
Import Type: Certificate  
Certificate Name: Cert-Star-ExampleCom  
Upload File: Generated Above

4. Import Key via PEM to f5  
Local Traffic > SSL Certificates > Cert-Star-ExampleCom  
Key > Import: Generated Abov

5. Create new SSL Profile  
Local Traffic > Profiles > SSL > Client > Create  
Name: SSL-ExampleCom  
Certificate: Cert-Star-ExampleCom  
Key: Cert-Star-ExampleCom

6. Update VirtualServer to use the new SSL Profile (Client SSL)
