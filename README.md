## CheetSheet

Hack the Box の攻略や、OSCP 取得のためのチートシートです。

悪用厳禁です。

## 目次


- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
- [WebScan](#Web_scan)
  - [Gobster](#Gobuster)
  - [Nikto](#Nikto)
  - [WPScan](#WPScan)
  - [BurpSuite](#BurpSute)
- [SMB Scan](#SMB_scan)
  - [smbclient](#smbclient)
  - [impacket](#impacket)
    - [impacket-smbserver](#impacket-smbserver)
  - [smbmap](#smbmap)
  - [enum4linux](#enum4linux)
- [侵入](#侵入)
  - [reverse_shell](#reverse_shell)
    - [Bash](#Bash)
    - [Powershell](#Powershell)
    - [Python](#Python)
    - [PHP](#PHP)
    - [Ruby](#Ruby)
    - [msfvenom_reverse_shell](#msfvenom_reverse_shell)
      - [Windows(meterpreter)](#Windows(meterpreter))
      - [Windows(netcatなど)](#Windows(netcatなど))
      - [Linux](#Linux)
      - [PHP(msfvenom)](#PHP(msfvenom))
      - [ASP(msfvenom)](#ASP(msfvenom))
      - [JSP(msfvenom)](#JSP(msfvenom))
      - [WAR(msfvenom)](#WAR(msfvenom))
      - [Python(msfvenom)](#Python(msfvenom))
      - [Bash(msfvenom)](#Bash(msfvenom))
      - [Perl(msfvenom)](#Perl(msfvenom))
      - [Handlers](#Handlers)
  - [Webアプリケーション](#Webアプリケーション)
      - [LFI](#lfi)
      - [XSS](#xss)
      - [SQLインジェクション](#sqlインジェクション)
        - [sqlmap](#sqlmap)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))
  - [Windows](#windows)
    - [windows-exploit-suggester](#windows-exploit-suggester)
    - [evilwinrm](#evilwinrm)
  - [Linux](#linux)
    - [linux-exploit-suggester](#linux-exploit-suggester)
    - [SUID](#SUID)
  - [HTTP/HTTPS Servers](#httpserver)
  - [Wordlist](#wordlist)
  - [Default Passwords](#default-passwords)
- [その他](#その他)
  - [MS17-010(EternalBlue)](#MS17-010(EternalBlue))
  - [ツールまとめ](#ツールまとめ)
    - [NetCat](#netcat)
    - [JohnTheRipper](#johntheripper)
    - [hydra](#hydra)
    - [searchsploit](#searchsploit)
    - [aircrack-ng](#aircrack-ng)
  - [未分類](#未分類)
    - [ssh](#ssh)
      - [sshトンネリング](#sshトンネリング)
      - [ssh-keygen](#ssh-keygen)
    - [ftp](#ftp)
    - [mysql](#mysql) 


# Network_scan

- Nmap

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

### 侵入

- reverse_shell
