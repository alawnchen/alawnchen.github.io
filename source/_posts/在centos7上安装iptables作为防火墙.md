---
title: 在centos7上安装iptables作为防火墙
date: 2018-01-30 07:15:00
tags: 
  - iptables
  - 防火墙
categories:
  - 技术
url: install-iptables-as-firewall-on-centos7
---

> 从centos7开始，centos就默认firewalld作为系统的防火墙。但在一些情况下，我们就不得不换回iptables。

<!--more-->

## 安装iptables并启动它

```
yum install iptables -y
yum update iptables  -y
yum install iptables-services -y
/usr/sbin/iptables
```

## 禁用自带的firewalld服务

```
systemctl stop firewalld
systemctl mask firewalld
```

## 设置iptables规则

```
iptables -P INPUT ACCEPT
iptables -F
iptables -X
iptables -Z
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -m state --state  RELATED,ESTABLISHED -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

## 保存iptables

```
service iptables save
```

## 启用iptables服务

```
systemctl enable iptables.service
systemctl start iptables.service
```

## 补充

如果想快速设置默认规则，也可以直接编辑/etc/sysconfig/iptables，并加入如下内容：

```
# Firewall configuration written by system-config-securitylevel
# Manual customization of this file is not recommended.
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:syn-flood - [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
-A INPUT -p icmp -m limit --limit 1/sec --limit-burst 10 -j ACCEPT
-A INPUT -f -m limit --limit 100/sec --limit-burst 100 -j ACCEPT
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j syn-flood
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A syn-flood -p tcp -m limit --limit 3/sec --limit-burst 6 -j RETURN
-A syn-flood -j REJECT --reject-with icmp-port-unreachable
COMMIT
```

保存后，启用iptables服务后后刷新重启iptables（先不要使用service iptables save保存，否则会被清空），然后重启系统。

```
systemctl enable iptables.service #centos7的用法
chkconfig --level 3 iptables on #centos7以下的用法
service iptables restart
reboot
```