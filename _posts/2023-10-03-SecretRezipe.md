---
layout: post
title: HTB{PC}
author: mr4ndr3w
tags: [write-up, hackthebox, challenge, zip, bkcrack]
---
<!--cut-->

* TOC
{:toc}

```text
┌──(kali㉿kali)-[~/Downloads/bkcrack-1.5.0-Linux]
└─$ ./bkcrack -L ../ingredients.zip                                                                 
bkcrack 1.5.0 - 2022-07-07
Archive: ../ingredients.zip
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 ZipCrypto  Store       49ba4df2           43           55 tmp/2939abb7c8ca1b699b904779ec930c5d/ingredients.txt
                                                                                                                     
┌──(kali㉿kali)-[~/Downloads/bkcrack-1.5.0-Linux]
└─$ ./bkcrack -C ../ingredients.zip -c tmp/2939abb7c8ca1b699b904779ec930c5d/ingredients.txt -p plain
bkcrack 1.5.0 - 2022-07-07
[07:44:32] Z reduction using 5 bytes of known plaintext
100.0 % (5 / 5)
[07:44:32] Attack on 1101708 Z values at index 6
Keys: 04ae88ef d5970258 2ab1731d
1.3 % (14537 / 1101708)
[07:44:48] Keys
04ae88ef d5970258 2ab1731d
                                                                                                                     
┌──(kali㉿kali)-[~/Downloads/bkcrack-1.5.0-Linux]
└─$ ./bkcrack -C ../ingredients.zip -c tmp/2939abb7c8ca1b699b904779ec930c5d/ingredients.txt -k 04ae88ef d5970258 2ab1731d -d secret.txt
bkcrack 1.5.0 - 2022-07-07
[07:46:03] Writing deciphered data secret.txt (maybe compressed)
Wrote deciphered data.
                                                                                                                     
┌──(kali㉿kali)-[~/Downloads/bkcrack-1.5.0-Linux]
└─$ ls    
bkcrack  example  license.txt  plain  readme.md  secret.txt  tools
                                                                                                                     
┌──(kali㉿kali)-[~/Downloads/bkcrack-1.5.0-Linux]
└─$ cat secret.txt       
Secret: HTB{C0mpr3sSi0n_1s_n0t_3NcryPti0n}
                                                                                                                     
```

