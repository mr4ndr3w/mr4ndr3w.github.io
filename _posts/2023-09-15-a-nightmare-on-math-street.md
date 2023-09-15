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
s.connect(("206.189.121.78",31432))
def checkFor(toEval,regex):
    while re.findall(regex,toEval):
        found=re.findall(regex,toEval)[0]
        toEval=toEval.replace(found,str(eval(found)),1)
    return toEval

def evaluate(content):
    while len(re.findall(r"[0-9]{1,99}",content))>1:
        if(re.findall(r"\([0-9*+ ]*\)",content)):
            nextContent=re.findall(r"\([0-9*+ ]*\)",content)[0]
            content=content.replace(nextContent,evaluate(nextContent[1:-1]))
        else:
            while len(re.findall(r"[0-9]{1,99}",content))>1:
                content=checkFor(content,r"[0-9]{1,99} \+ [0-9]{1,99}")
                content=checkFor(content,r"[0-9]{1,99} \* [0-9]{1,99}")
    return content

for nb in range(500):
    print(f"{nb+1}/500")
    reply = str(s.recv(4096))
    toEval=re.findall(r"]: (.+)? =",reply)[0]
    toEval=evaluate(toEval)
    result=eval(toEval)
    s.sendall(bytes(f"{result}\n","utf-8"))

flag = str(s.recv(4096))
print(flag)
```
