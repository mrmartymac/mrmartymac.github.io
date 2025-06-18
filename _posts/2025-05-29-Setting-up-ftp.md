---
title: Setting up VSFTP
date: 2025-05-29 07:21:50 -400
categories: [Cheatsheet, FTP]
tags: [ftp, vstp] # TAG names should always be lowercase
author: mm
---

846  systemctl status ncftpd
  848  firewall-cmd --add-service=ftp --permanent --zone=public
  853  ln -s /usr/lib/systemd/system/vsftpd.service /etc/systemd/system/multi-user.target.wants/vsftpd.service 
  854  ln -s /etc/systemd/system/multi-user.target.wants/vsftpd.service /usr/lib/systemd/system/vsftpd.service
  857  systemctl restart vsftpd
  866  more ftp-custom.xml 
  868  more /etc/vsftpd/vsftpd.conf
  869  nano /etc/vsftpd/vsftpd.conf
  870  systemctl restart vsftpd
  871  nano /etc/vsftpd/vsftpd.conf
  877  systemctl status vsftpd
  879  nano /etc/vsftpd/vsftpd.conf
  880  systemctl restart vsftpd
  881  passwd csn_ftp
  882  cd /etc/vsftpd/
  884  nano vsftpd.conf
  885  systemctl restart vsftpd
  887  cd /u/ftp/
  899  cd /u/ftp/CSN/
  947  clear ; ftp
 1004  cd ftp
 1014  nano /etc/vsftpd/vsftpd.conf
 1015  nano /etc/vsftpd/user_list 
 1016  history | grep csn_ftp
 1017  useradd -c "SCS FTP" -d /u/ftp/SCS -m -s /bin/ftponly scs_ftp
 1018  mkdir /u/ftp/SCS
 1019  chown scs_ftp:scs /u/ftp/SCS
 1020  passwd scs_ftp
 1022  cd /u/ftp/
 1033  deluser scs_ftp
 1034  rmuser scs_ftp
 1036  userdel -r scs_ftp
 1043  nano /etc/vsftpd/user_list 