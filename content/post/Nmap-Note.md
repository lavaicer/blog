---
title: "Nmap Note"
date: 2022-03-18T17:18:02+08:00
draft: false
weight: 9
categories: ["安全工具"]
tags: ["安全工具", "Nmap"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---
-sn	只用 ping 扫描，不扫描端口
--data-length 25	添加随机数据的数据包
-traceroute		路由跟踪 ip 协议

## 扫描范围的确定

### 对连续范围内的主机进行扫描

> Nmap [IP 地址的范围]
> `nmap 192.168.43.1-255`

### 对整个子网进行扫描/

> Nmap [IP 地址 / 掩码位数]
> `nmap 192.168.43.1/24`

### 对多个不连续的主机进行扫描

> Nmap [ip1 ip2 ip3 ……]

### 扫描时排除指定目标

> Nmap [目标] --exclude [目标]
> `nmap 192.168.43.1/24 --exclude 192.168.43.86`

### 对一个文本中的地址列表进行扫描

> Nmap -iL [文本文件]
> `nmap -iL list.txt`

### 随机确定扫描目标

> Nmap -iR [目标的数量]
> `nmap -iR 3`	# 随机在互联网上找三个扫描

# 活跃主机发现

## 基于 ARP 协议

> Nmap -PR [目标]

## 基于 ICMP 协议

### 响应请求和应答

> Nmap -PE [目标]

### 时间戳请求

> Nmap -PP [目标]

### 地址掩码请求和应答

> Nmap -PM [目标]

## 基于 TCP 协议

常见 TCP 扫描端口

| 端口号 |   提供的服务   |
| :----: | :------------: |
|   80   |      http      |
|   25   |      smtp      |
|   22   |      ssh      |
|  443  |     https     |
|   21   |      ftp      |
|  113  |      auth      |
|   23   |     telnet     |
|   53   |     domain     |
|  554  |      rtsp      |
|  3389  | ms-term-server |
|  1723  |      pptp      |
|  389  |      ldap      |
|  636  |    ldapssl    |
|  256  | fwl-securemote |

### TCP SYN 扫描

> Nmap -PS [port1,port1……] [目标]

### TCP ACK 扫描

> Nmap -PA [目标]

## 基于 UDP 协议

> Nmap -PU [目标]

注：TCP 扫描要扫开放的端口，UDP 扫描得是目标主机关闭的端口，在扫描过程中需要避开常用的 UDP 协议端口 例如：DNS（53），SNMP（161）

## 基于 SCTP 协议

TCP 	单地址连接	协议基于字节流	只能支持一个流		三次握手
SCTP	多地址连接	协议基于消息流	可同时支持多个流	四次握手

> Nmap -PY [目标]

## 与 DNS 相关选项

### IP 反查域名

> Nmap -R [IP]

### 对于已经知道的主机的域名，不进行转换。提高效率

> Nmap -n -R [目标 IP]

### 不想在自己的 DNS 服务器留下查询记录，可以指定服务器

> Nmap -R --dns-servers 202.99.160.68 211.81.200.8

### 观察 Nmap 发出的数据包

> --packet-trace

# 端口扫描技术

## 端口的分类

### 公认端口

也称常用端口，0 ~ 1024

### 注册端口

1025 ~ 49151，他们通常也会关联到一些服务上，但是并没有明确的规定，不同的程序可根据实际情况进行定义

### 动态和/或私有端口

49152 ~ 65535，常见的服务不会使用，但因为这部分端口不容易引起注意，因此有些程序十分中意

## Nmap 中对端口状态的定义

- open：有程序接受 TCP 连接或 UDP 报文
- closed：不意味着没有反应，相比较而言，没有程序在 open 上监听
- filtered：存在目标网络数据包过滤，导致 Nmap 无法确定该端口是否开放
- unfiltered:表明目标端口可以访问，但是 Nmap 无法判断是 open 还是 closed。通常只有在进行 ACK 扫描是才会出现
- open | filtered：无法确定端口是开放的还是被过滤了，开放的端口不响应就是一个例子
- closed | filtered：无法确定端口是关闭的还是被过滤了。只有使用 idle 扫描时才会出现

## Nmap 中端口扫描技术

**TCP连接扫描**

使用操作系统的网络连接系统调用 connect()，对目标主机发起 TCP 三路握手，待完成后 Nmap 立即中断此次连接。

Nmap 通过获取每个尝试连接的状态信息来判定侦测端口的状态

**SYN扫描**

Nmap 产生一个 SYN 数据报文，如果侦测端口开放并返回 SYN-ACK 响应报文

Nmap 据此发送 RST 报文给侦测端口结束当前连接，这样做的好处在于缩短了端口扫描时间

**UDP扫描**

UDP 本身是无连接的协议，Nmap 向目标主机的端口发送 UDP 探测报文

如果端口没有开放，被侦测主机将会发送一个 ICMP 端口不可到达的消息

Nmap 根据这个消息确定端口闭合（closed）或者被过滤 (unfiltered)

通常没有回复意味着端口是开放（open）状态 。

**ACK扫描**

这种扫描比较特殊，它不能确切知道端口的基本状态，而是主要用来探测防火墙是否存在以及其中设定的过滤规则

**FIN扫描**

和 SYN 扫描相比，这种方式更为隐蔽，因此能够穿过防火墙的过滤

关闭（closed）端口将会返回合适的 RST 报文，而开放端口将忽略这样的侦测报文

具备类似防火墙不敏感特性的还有 -sN NULL 扫描，-sX X-mas 扫描。

### SYN

> Nmap -sS [target]

### Connect

> Nmap -sT [target]

### UDP

> Nmap -sU [target]

### TCP FIN

> Nmap -sF [target]

### NULL

不含任何标志的数据包

> Nmap -sN [target]

### Xmas Tree

向目标端口发一个含有 FIN、URG、PUSH 标志的数据包

> Nmap -sX [target]

### idle

借助第三方，根据 ip id 的增长来判断

## 指定扫描的端口

|        指定端口        |           指定端口的选项           |
| :--------------------: | :--------------------------------: |
|   扫描常见的100端口   |          `-F [target]`          |
|     指定某一个端口     |           `-p [port]`           |
| 使用名字来指定扫描端口 |           `-p [name]`           |
| 使用协议来指定扫描端口 | `-p U:[UDP ports],T:[TCP ports]` |
|      扫描所有端口      |              `-p *`              |
|      扫描常用端口      |      `--top-ports [number]`      |

# 操作系统与服务检测技术

|        选项        |                                    意义                                    |
| :----------------: | :-------------------------------------------------------------------------: |
| `--osscan-limit` | 只要满足“同时具有状态为 open 和 closed 的端口” 条件的主机进行操作系统检测 |
| `--osscan-guess` |                    猜测认为最接近目标的匹配操作系统类型                    |
| `--max-retries` |                          对操作系统检测尝试的次数                          |

## 操作系统指纹扫描

## 使用 Nmap 进行服务发现

- `-sV` 打开版本探测，也可用 `-A` 同时打开操作系统和服务发现
- `--allports` 不为版本探测排除任何端口
- `--version-intensity<intensity>` 设置版本扫描强度
- `--version-light` 打开轻量级模式
- `--version-all` 尝试每个探测
- `--version-trace` 跟踪版本扫描活动
- `sR` RPC 扫描

# Nmap 高级技术与防御措施

## Nmap 的伪装技术

### 报文分段

> Nmap -f [target]

### 使用指定的 MTU（最大传输单元）

> Nmap --mtu [target]

### 使用诱饵主机隐蔽扫描

`-D <decoy1 [, decoy2][, ME], ……>`

### 源端口欺骗

- `--source-port <portnumber>`
- `-g <portnumber>`

### 源IP欺骗

- `nmap -sI www.0day.com:80 192.168.0.8`

### 发送报文时附加随机数据

`--data-length <number>`

### 设置 IP time-to-live 域

`--ttl <value>`

### MAC 地址欺骗

`--spoof-mac <mac address, prefix, or vendor name>`

- 0：随即一个 Mac 地址
- 使用分好分割的十六进制偶数：当做 Mac 地址
- 小于12的十六进制数字，Nmap会随机填充剩下的6字节
- 不是0或十六进制字符串 ==> 使用厂商的OUI（3字节前缀，然后随机填充剩余的三个字节）

## TCP Connect 扫描的检测

> Nmap -sT [target]

工具： Wireshark 的使用

## Nmap 格式化输出

> Nmap -oN [\*.txt] [target]
> Nmap -oX [\*.xml] [target]
> Nmap -oG [\*.grep] [target]

# Nmap 脚本的使用

```bash
nmap --script 类别
```

```verilog
- auth: 负责处理鉴权证书（绕开鉴权）的脚本  
- broadcast: 在局域网内探查更多服务开启状况，如dhcp/dns/sqlserver等服务  
- brute: 提供暴力破解方式，针对常见的应用如http/snmp等  
- default: 使用-sC或-A选项扫描时候默认的脚本，提供基本脚本扫描能力  
- discovery: 对网络进行更多的信息，如SMB枚举、SNMP查询等  
- dos: 用于进行拒绝服务攻击  
- exploit: 利用已知的漏洞入侵系统  
- external: 利用第三方的数据库或资源，例如进行whois解析  
- fuzzer: 模糊测试的脚本，发送异常的包到目标机，探测出潜在漏洞
- intrusive: 入侵性的脚本，此类脚本可能引发对方的IDS/IPS的记录或屏蔽
- malware: 探测目标机是否感染了病毒、开启了后门等信息  
- safe: 此类与intrusive相反，属于安全性脚本  
- version: 负责增强服务与版本扫描（Version Detection）功能的脚本  
- vuln: 负责检查目标机是否有常见的漏洞（Vulnerability），如是否有MS08_067
```

一个常用命令

```shell
./nmap -sT -sV -p 21,80,443,873,2601,2604,3128,4440,6082,6379,8000,8008,8080,8081,8090,8099,8088,8888,9000,9090,9200,11211,27017,28017 --max-hostgroup 10 --max-parallelism 10 --max-rtt-timeout 1000ms --host-timeout 800s --max-scan-delay 2000ms -iL iplist.txt -oN result/port.txt --open
```

Nmap 扫描策略

```

# 适用所有大小网络最好的 nmap 扫描策略
# 主机发现，生成存活主机列表
$ nmap -sn -T4 -oG Discovery.gnmap 192.168.56.0/24
$ grep "Status: Up" Discovery.gnmap | cut -f 2 -d ' ' > LiveHosts.txt

# 端口发现，发现大部分常用端口
# http://nmap.org/presentations/BHDC08/bhdc08-slides-fyodor.pdf
$ nmap -sS -T4 -Pn -oG TopTCP -iL LiveHosts.txt
$ nmap -sU -T4 -Pn -oN TopUDP -iL LiveHosts.txt
$ nmap -sS -T4 -Pn --top-ports 3674 -oG 3674 -iL LiveHosts.txt

# 端口发现，发现全部端口，但 UDP 端口的扫描会非常慢
$ nmap -sS -T4 -Pn -p 0-65535 -oN FullTCP -iL LiveHosts.txt
$ nmap -sU -T4 -Pn -p 0-65535 -oN FullUDP -iL LiveHosts.txt

# 显示 TCP\UDP 端口
$ grep "open" FullTCP|cut -f 1 -d ' ' | sort -nu | cut -f 1 -d '/' |xargs | sed 's/ /,/g'|awk '{print "T:"$0}'
$ grep "open" FullUDP|cut -f 1 -d ' ' | sort -nu | cut -f 1 -d '/' |xargs | sed 's/ /,/g'|awk '{print "U:"$0}'

# 侦测服务版本
$ nmap -sV -T4 -Pn -oG ServiceDetect -iL LiveHosts.txt

# 扫做系统扫描
$ nmap -O -T4 -Pn -oG OSDetect -iL LiveHosts.txt

# 系统和服务检测
$ nmap -O -sV -T4 -Pn -p U:53,111,137,T:21-25,80,139,8080 -oG OS_Service_Detect -iL LiveHosts.txt
```

Nmap 躲避防火墙

```

# 分段
$ nmap -f

# 修改默认 MTU 大小，但必须为 8 的倍数(8,16,24,32 等等)
$ nmap --mtu 24

# 生成随机数量的欺骗
$ nmap -D RND:10 [target]

# 手动指定欺骗使用的 IP
$ nmap -D decoy1,decoy2,decoy3 etc.

# 僵尸网络扫描, 首先需要找到僵尸网络的IP
$ nmap -sI [Zombie IP] [Target IP]

# 指定源端口号
$ nmap --source-port 80 IP

# 在每个扫描数据包后追加随机数量的数据
$ nmap --data-length 25 IP

# MAC 地址欺骗，可以生成不同主机的 MAC 地址
$ nmap --spoof-mac Dell/Apple/3Com IP
```

WEB 漏洞扫描

```

cd /usr/share/nmap/scripts/
wget http://www.computec.ch/projekte/vulscan/download/nmap_nse_vulscan-2.0.tar.gz && tar xzf nmap_nse_vulscan-2.0.tar.gz
nmap -sS -sV --script=vulscan/vulscan.nse target
nmap -sS -sV --script=vulscan/vulscan.nse –script-args vulscandb=scipvuldb.csv target
nmap -sS -sV --script=vulscan/vulscan.nse –script-args vulscandb=scipvuldb.csv -p80 target
nmap -PN -sS -sV --script=vulscan –script-args vulscancorrelation=1 -p80 target
nmap -sV --script=vuln target
nmap -PN -sS -sV --script=all –script-args vulscancorrelation=1 target
```

USERPASS_FILE => /usr/share/wordlists/metasploit/piata_ssh_userpass.txt
