---
title: "Iptables"
date: 2022-04-05T15:33:54+08:00
draft: false
weight: 9
categories: ["WAF"]
tags: ["WAF"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---

规则链

> 规则的作用：对数据包进行过滤或处理
> 链的作用：容纳各种防火墙规则
> 链的分类依据：处理数据包的不同时机

默认 5 种规则链

> INPUT： 处理入站数据包
> OUTPUT： 处理出站数据包
> FORWARD： 处理转发数据包
> POSTROUTING： 在进行路由选择后处理数据包
> PREROUTING： 在进行路由选择前处理数据包

规则表

> 表的作用： 容纳各种规则链
> 表的划分依据： 防火墙规则的作用相似

默认包括 4 个规则表

> raw 表： 确定是否对该数据包进行状态跟踪
> mangle 表： 为数据包设置标记
> nat 表： 修改数据包中的源、目的 IP 地址或端口
> filter 表： 确定是否放行该数据包（过滤）

![](https://raw.githubusercontent.com/lavaicer/Img/main/202204051611486.png)

规则表之间的顺序

> raw -> mangle -> nat -> filter

规则链之间的顺序

> 入站： PREROUTING -> INPUT
> 出站： OUTPUT -> POSTROUTING
> 转发： PREROUTING -> FORWARD -> POSTROUTING

规则链内的匹配顺序

> 按顺序依次排查，匹配即停止（LOG 策略例外）
> 若找不到相匹配的规则，则按该链的默认策略处理

![](https://raw.githubusercontent.com/lavaicer/Img/main/202204051800113.png)

iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型]
`iptables -t filter -I INPUT -p icmp -j REJECT`

注意：

> 不指定表名时，默认指 filter 表
> 不指定链名时， 默认指表内的所有链
> 除非设置链的默认策略，否则必须指定匹配条件
> 选项、链名、控制类型使用大写字母，其余为小写

数据包的常见控制类型

> ACCEPT: 允许通过
> DROP: 直接丢弃，不给任何回应
> REJECT: 拒绝通过，必要时会给出提示
> LOG: 记录日志信息，然后传给下一条规则继续匹配

添加新规则：

> -A： 在链的末尾追加一条规则
> -I： 在链的开头（或指定序号）插入一条规则

查看规则列表：

> -L： 列出所有规则条目
> -n： 以数字形式显示地址、端口等信息
> -v： 以更详细的方式显示规则信息
> --line-numbers: 查看规则时，显示规则的序号

删除、清除规则：

> -D: 删除链内指定序号（或内容）的一条规则
> -F: 清空所有的规则

设置默认策略：

> -P: 为指定的链设置默认规则

![](https://raw.githubusercontent.com/lavaicer/Img/main/202204052151700.png)

状态：

> NEW: 新建连接
> ESTABLISHED: 已有相关连接
> RELATED: 相关连接其他进程
> INVALID： 无法识别
> UNTRACKED： 无法连接

规则的匹配条件

- 通用匹配
  可直接使用，不依赖于其他条件或扩展
  包括网络协议、IP 地址、网络接口等条件

- 隐含匹配
  要求以特定的协议匹配作为前提
  包括端口、TCP 标记、ICMP 类型等条件

- 显示匹配
  要求以 “-m 扩展模块” 的形式明确指出类型
  包括多端口、MAC 地址、IP 范围、数据包状态等条件

常见的通用匹配条件：
协议匹配： -p 协议名
地址匹配： -s 源地址、 -d 目的地址
接口匹配： -i 入站网卡、 -o 出站网卡

用的隐含匹配条件：
端口匹配： --sport 源端口、 --dport 目的端口
TCP 标记匹配： --tcp-flags 检查范围被设置的标记
ICMP 类型匹配：--icmp-type ICMP 类型

常用的显式匹配条件：
多端口匹配：
-m multiport--sports 源端口列表
-m multiport--dports 目的端口地址

IP 范围匹配： -m iprange--src-range IP 范围
MAC 地址匹配： -m mac--mac-source MAC 地址
状态匹配： -m state--state 连接状态

![](https://raw.githubusercontent.com/lavaicer/Img/main/202204061339493.png)

