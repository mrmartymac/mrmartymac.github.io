---
title: OpenVPN on Linode
date: 2024-09-12 12:52:37 -400
categories: [Cheatsheets, Linode]
tags: [linode, openvpn] # TAG names should always be lowercase
author: mm
---
## Creating your account

Open the browser you use to authenticate and work in Google Drive/Gmail.
Go to any "Google page" like the search, Gmail, or Drive.
1. Click on the Google Apps icon in the top left of the screen.
2. Scroll down and choose the **OpenVPN on Linode** app from the list near the bottom.  
![Open Google App](/images/openvpn-linode/OpenVPN-Linode1.png)
3. Authenticate using your Google Account.  
![Authenticate](/images/openvpn-linode/OpenVPN-Linode2.png)
4. The instance does not have a certificate, you will have to allow your browser to proceed. Click on Advanced
5. Click on `Proceed to 45.56.105.91 (unsafe).` I promise it is ok.  
![Missing Certificate](/images/openvpn-linode/OpenVPN-Linode3.png)
6. As I expect you already have an OpenVPN client, you only need to download the **Connection profile**
![Download Profile](/images/openvpn-linode/OpenVPN-Linode4.png)
7. You should now open your VPN application and import the new profile you just downloaded.

## Admin Notes
Login URL
[OpenVPN Dashboard on Linode](https://45-56-105-91.ip.linodeusercontent.com:943/admin/status_overview)