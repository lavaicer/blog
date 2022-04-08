---
title: "Linux 常用命令"
date: 2022-03-18T17:03:16+08:00
draft: true
weight: 9
categories: ["linux", "搬运工"]
tags: ["linux", "bash"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---

## 基础命令

du 计算目录容量
grep 正则匹配，搜索文本
ln -s 源文件 目标文件(-s 软链接、不可删除源文件；硬链接时，源文件只能为文件不能是目录)
wc 显示文件的行、单词、字节统计信息（-l、-w、-c）

## 账户管理

id #打印用户的 UID 和 GID，
root 组的 GID 号是：0，bin 组 GID 号是：1，daemon 组 GID 号是：2，sys 组 GID 号是：3
passwd #后单跟用户名跟改密码
-l #锁定用户不能跟改密码
-d #清除用户密码
-S #查询用户密码状态

who #查看当前登录用户（tty 本地登陆 pts 远程登录）
w #查看系统信息，想知道某一时刻用户的行为
uptime #查看登陆多久、多少用户，负载

useradd #添加用户，-d 指定家目录，-g 指定主要组，-G 指定次要组，-s 指定缺省 shell
groupadd #添加组，-g 指定组 ID
usermod #修改用户信息，-e 有效期，-f 宽限天数，-l 账户名称，-L 锁定用户，-u 修改用户 ID
userdel #删除用户，-f 强制删除 即使用户已登录，-r 同时删除用户相关所有文件
groupdel #删除工作组

#账户信息文件：/etc/passwd
root❌0:0:root:/root:/bin/bash
用户名：密码：用户 ID：组 ID：用户说明(描述)：用户主(家)目录：缺省 shell(登陆后的 shell)
注意：无密码只允许本机登陆，远程不允许登陆

#账户密码文件：/etc/shadow
root:$Gs1qhL2p3ZetrE4.kMHx6qgbTcjQSt.Ft7ql1WpkopY/:16809:0:99999:7:::
用户名：加密密码：密码最后一次修改日期：两次密码的修改时间间隔：
密码有效期：密码到期的警告天数：密码过期宽限天数：账号失效时间：保留

#组账户信息文件：/etc/group
root❌0:
组名：口令：组标识号：组内用户列表
