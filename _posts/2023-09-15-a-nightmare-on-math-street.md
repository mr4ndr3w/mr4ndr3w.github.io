---
layout: post
title: HTB{A Nightmare On Math Street}
author: mr4ndr3w
tags: [write-up, hackthebox, challenge, misc]
---
<!--cut-->

* TOC
{:toc}

```text
import socket 
import re   


s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("206.189.24.162",30351))

def check(calc, regex):
    while re.findall(regex, calc):
        found= re.findall(regex, calc)[0]
        print(found)
        calc = calc.replace(found, str(eval(found)),1)
        print(calc)
    return calc


def calculate(line):
    while len(re.findall(r"[0-9]{1,99}",line))>1:
        if(re.findall(r"\([0-9*+ ]*\)",line)):
            nextline=re.findall(r"\([0-9*+ ]*\)",line)[0]
            print(nextline)
            line = line.replace(nextline,calculate(nextline[1:-1])) 
            print(line)
        else:
            while len(re.findall(r"[0-9]{1,99}",line))>1:
                line=check(line, r"[0-9]{1,99} \+ [0-9]{1,99}")
                line=check(line, r"[0-9]{1,99} \* [0-9]{1,99}")
    return line




for line in range(500):
    print(f"{line+1}/500")
    reply = str(s.recv(4096))
    calc = re.findall(r"]: (.+)? =",reply)[0]
    calc = calculate(calc)
    result=eval(calc)
    s.sendall(bytes(f"{result}\n","utf-8"))



flag = str(s.recv(4096))

print(flag)
```
