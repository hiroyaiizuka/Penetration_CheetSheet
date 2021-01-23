## CheetSheet

Hack the Box の攻略や、OSCP 取得のためのチートシートです。

悪用厳禁です。

## 目次

### Network Scan

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
