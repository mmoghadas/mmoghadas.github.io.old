---
title: Enabled WebVPN on Cisco routers (AnyConnect)
author: Mike Moghadas
layout: post
categories:
  - Cisco
  - Exchange
tags:
  - cisco vpn
  - ssl
  - ssl vpn
  - webvpn
---
<p><strong>Upload AnyConnect Packages</strong><br />
anyconnect-win-2.4.1012-k9.pkg<br />
anyconnect-macosx-i386-2.4.1012-k9.pkg<br />
anyconnect-linux-2.4.1012-k9.pkg</p>
<p>1. On the router</p>
<p><!--more--></p>
<p>{% codeblock lang:objc %}<br />
router.od100a#mkdir flash:/webvpn<br />
{% endcodeblock %}</p>
<p>2. Use tftp to upload the following packages to flash:/webvpn<br />
{% codeblock lang:objc %}<br />
copy tftp://10.210.15.103/anyconnect-win-2.4.1012-k9.pkg flash:/webvpn/<br />
copy tftp://10.210.15.103/anyconnect-macosx-i386-2.4.1012-k9.pkg flash:/webvpn/<br />
copy tftp://10.210.15.103/anyconnect-linux-2.4.1012-k9.pkg flash:/webvpn/<br />
{% endcodeblock %}</p>
<p>3. Create SelfSigned Certificate<br />
{% codeblock lang:objc %}<br />
conf t<br />
crypto pki trustpoint VPOD-SelfSigned-SSL<br />
enrollment selfsigned<br />
rsakeypair VPOD-SelfSigned-SSL 1024 1024<br />
end<br />
Router#conf t<br />
crypto pki enroll VPOD-SelfSigned-SSL</p>
<p>% Include the router serial number in the subject name? [yes/no]: yes<br />
% Include an IP address in the subject name? [no]: yes<br />
Enter Interface name or IP Address[]: FastEthernet0/0<br />
Generate Self Signed Router Certificate? [yes/no]: yes<br />
end<br />
{% endcodeblock %}</p>
<p>4. Configure WebVPN<br />
{% codeblock lang:objc %}<br />
conf t<br />
webvpn gateway VPOD-WebVPN<br />
ip address 208.77.x.x port 443<br />
ssl trustpoint VPOD-SelfSigned-SSL<br />
inservice<br />
webvpn install svc flash:/webvpn/anyconnect-win-2.4.1012-k9.pkg sequence 1<br />
webvpn install svc flash:/webvpn/anyconnect-macosx-i386-2.4.1012-k9.pkg sequence 2<br />
webvpn install svc flash:/webvpn/anyconnect-linux-2.4.1012-k9.pkg sequence 3<br />
webvpn context Admins<br />
title “VPOD-WebVPN for example Administrators”<br />
secondary-color #9ABEDC<br />
title-color #4186BE<br />
ssl authenticate verify all</p>
<p>url-list “AdminLinks”<br />
heading “AdminLinks”<br />
url-text “v1″ url-value “http://x.example.com”<br />
url-text “v2″ url-value “http://y.example.com”<br />
exit<br />
policy group GPAdmins<br />
url-list “AdminLinks”<br />
functions svc-enabled<br />
svc address-pool “ravpn-pool-1″<br />
svc default-domain “vpod.example.net”<br />
svc keep-client-installed<br />
svc rekey method new-tunnel<br />
svc split include 172.10.10.0 255.255.255.0<br />
svc dns-server primary 208.67.220.220<br />
svc dns-server secondary 208.67.222.222<br />
exit<br />
default-group-policy GPAdmins<br />
aaa authentication list vpn-xauth-1<br />
gateway VPOD-WebVPN<br />
inservice<br />
{% endcodeblock %} </p>
