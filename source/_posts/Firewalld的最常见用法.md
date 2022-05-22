---
title: Firewalld的最常见用法
date: 2022-05-22 02:37:38
tags: 
  - Firewalld
  - 防火墙
  - ipset
  - 网络安全
categories:
  - 技术
photos:
  - "/images/9644d159a79fb11b4a8416cc72a1bdc44fcb718c-fp-2000-500-0-0.jpeg"
url: the-most-common-usage-firewalld
description: 7.0之后，centos自带Firewalld作为默认的防火墙软件。而常见的Linux服务器操作系统里面也有不小比例的发行版本也自带了Firewalld作为防火墙软件。所以呢，对Firewalld有些基本了解还是需要的。
---

> 7.0之后，centos自带Firewalld作为默认的防火墙软件。而常见的Linux服务器操作系统里面也有不小比例的发行版本也自带了Firewalld作为防火墙软件。所以呢，对Firewalld有些基本了解还是需要的。这里以centos8为例子，说一说一些Firewalld用法。

## 启用Firewalld

默认情况下，centos8的Firewalld的服务是处于屏蔽状态。我们需要先将Firewalld服务解除屏蔽，再启用相关服务，最后启动服务。

```
systemctl unmask firewalld.service
systemctl enable firewalld.service
systemctl start firewalld.service
```

## 查看Firewalld运行状态

```
firewall-cmd --state
```

如果显示“running”，说明Firewalld已经启动。

## 查看Firewalld激活区域

```
firewall-cmd --get-active-zone
```

默认启用了public区域，接下来的例子都是以public区域为例子。

## 添加服务

默认情况下，没有添加http和https服务，所以网站无法正常访问。所以，我们需要手动添加http和https服务。可以一条条地添加，也可以批量添加。

```
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
//可以批量添加
firewall-cmd --zone=public --add-service=http --add-service=https --permanent
```

## 移除服务

ipv6 DHCP服务一开始就被添加到服务里面。一般都不需要，可以通过下面的命令移除。还有cockpit，是一个监控软件，也可以删除。

```
firewall-cmd --zone=public --remove-service=dhcpv6-client --permanent
firewall-cmd --zone=public --remove-service=cockpit --permanent
//可以批量移除
firewall-cmd --zone=public --remove-service=dhcpv6-client --remove-service=cockpit --permanent
```

## 添加端口

```
firewall-cmd --zone=public --add-port=22222/tcp --permanent
firewall-cmd --zone=public --add-port=22223/udp --permanent
firewall-cmd --zone=public --add-port=22224-22229/tcp --permanent
//可以批量添加
firewall-cmd --zone=public --add-port=22222/tcp --add-port=22223/udp --add-port=22224-22229/tcp --permanent
```

其中的tcp和udp是协议。

## 移除端口

```
firewall-cmd --zone=public --remove-port=22222/tcp --permanent
firewall-cmd --zone=public --remove-port=22223/udp --permanent
firewall-cmd --zone=public --remove-port=22224-22229/tcp --permanent
//可以批量移除
firewall-cmd --zone=public --remove-port=22222/tcp --remove-port=22223/udp --remove-port=22224-22229/tcp --permanent
```

## 添加来源

可以是IP地址，也可以是IP地址范围

```
firewall-cmd --zone=public --add-source=10.10.10.1 --permanent
firewall-cmd --zone=public --add-source=10.10.10.0/24 --permanent
//可以批量添加
firewall-cmd --zone=public --add-source=10.10.10.1 --add-source=10.10.10.0/24 --permanent
```

## 移除来源

```
firewall-cmd --zone=public --remove-source=10.10.10.1 --permanent
firewall-cmd --zone=public --remove-source=10.10.10.0/24 --permanent
//可以批量移除
firewall-cmd --zone=public --remove-source=10.10.10.1 --remove-source=10.10.10.0/24 --permanent
```

## 添加端口转发

```
//将tcp协议流量从8080端口转发到内部的22222端口（同一机器）
firewall-cmd --zone=public --add-forward-port="port=8080:proto=tcp:toport=22222" --permanent
//将tcp协议流量从8080端口转发到外部的22222端口（不同机器，转发到10.10.10.2，需要先开启masquerade，伪装流量）
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --zone=public --add-forward-port="port=8080:proto=tcp:toport=22222:toaddr=10.10.10.2" --permanent
```

## 移除端口转发

```
//对应上面的同一机器转发
firewall-cmd --zone=public --add-forward-port="port=8080:proto=tcp:toport=22222" --permanent
//对应上面的不同机器转发
firewall-cmd --zone=public --remove-forward-port="port=8080:proto=tcp:toport=22222:toaddr=10.10.10.2" --permanent
firewall-cmd --zone=public --remove-masquerade --permanent
```

## ipset

```
//新增一个名称为leochan，hash:ip类型的ipset
firewall-cmd --permanent --new-ipset=leochan --type='hash:ip'
//新增一个名称为leochan，hash:ip类型的ipset，IP是ipv6
firewall-cmd --permanent --new-ipset=leochan --type='hash:ip' --option='family=inet6'
//查看ipset支持的类型
firewall-cmd --get-ipset-types
//移除掉名称为leochan的ipset
firewall-cmd --permanent --delete-ipset=leochan
//显示名称为leochan的ipset对应的文件
firewall-cmd --permanent --path-ipset=leochan
//往名称为leochan的ipset添加10.10.10.10
firewall-cmd --permanent --ipset=leochan --add-entry=10.10.10.10
//从名称为leochan的ipset移除10.10.10.10
firewall-cmd --permanent --ipset=leochan --remove-entry=10.10.10.10
//在名称为leochan的ipset查询10.10.10.10是否存在其中
firewall-cmd --permanent --ipset=leochan --query-entry=10.10.10.10
```

ipset类型说明

hash:ip 可以存储IP地址
hash:ip,mark 可以存储IP地址和网络掩码对
hash:ip,port 可以存储IP地址和端口号对
hash:ip,port,ip 可以存储IP地址、端口号和IP地址三元组
hash:ip,port,net 可以存储IP地址、端口号和网络号三元组
hash:mac 可以存储MAC地址
hash:net 可以存储网络号
hash:net,iface 可以存储网络号和网络接口对
hash:net,net 可以存储网络号和网络号对
hash:net,port 可以存储网络号和端口号对
hash:net,port,net 可以存储网络号、端口号和网络号三元组

## 添加富规则

注意不同机器端口转发前，必须启用流量伪装

```
firewall-cmd --zone=public --add-masquerade --permanent
```

```
//允许来自10.10.10.10的ipv4流量
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.10" accept' --permanent
//允许来自10.10.10.10请求22222端口tcp协议的ipv4流量
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.10" port port="22222" protocol="tcp" accept' --permanent
//拒绝来自10.10.10.10请求22222端口tcp协议的ipv4流量
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.10" port port="22222" protocol="tcp" reject' --permanent
//允许来自10.10.10.10请求22222端口tcp协议的ipv4流量，并将其转发到22223端口（同一机器内转发）
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.10" forward-port port="22222" protocol="tcp" to-port="22223" accept' --permanent
//允许来自10.10.10.10请求22222端口tcp协议的ipv4流量，并将其转发到10.10.10.3的22223端口（不同机器内转发）
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.10" forward-port port="22222" protocol="tcp" to-port="22223" to-addr="10.10.10.3" accept' --permanent
//允许从mac为00:11:22:33:44:55的客户端访问ssh服务
firewall-cmd --zone=public --add-rich-rule='rule source mac="00:11:22:33:44:55" service name="ssh" accept' --permanent
//允许从mac为00:11:22:33:44:55的客户端，通过ipv4访问ssh服务
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source mac="00:11:22:33:44:55" service name="ssh" accept' --permanent
//允许从mac为00:11:22:33:44:55的客户端请求22222端口tcp协议
firewall-cmd --zone=public --add-rich-rule='rule source mac="00:11:22:33:44:55" port port="22222" protocol="tcp" accept' --permanent
//允许从mac为00:11:22:33:44:55的客户端，通过ipv4请求22222端口tcp协议
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source mac="00:11:22:33:44:55" port port="22222" protocol="tcp" accept' --permanent
//允许存储在名称为leochan的ipset里面的机器，通过ipv4请求22222端口tcp协议
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source ipset="leochan" port port="22222" protocol="tcp" accept' --permanent
//阻止存储在名称为leochan的ipset里面的机器，通过ipv4请求22222端口tcp协议
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source ipset="leochan" port port="22222" protocol="tcp" reject' --permanent
```

## 移除富规则

下面的移除命令和上面的添加命令一一对应

```
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="10.10.10.10" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="10.10.10.10" port port="22222" protocol="tcp" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="10.10.10.10" port port="22222" protocol="tcp" reject' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="10.10.10.10" forward-port port="22222" protocol="tcp" to-port="22223" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="10.10.10.10" forward-port port="22222" protocol="tcp" to-port="22223" to-addr="10.10.10.3" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule source mac="00:11:22:33:44:55" service name="ssh" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source mac="00:11:22:33:44:55" service name="ssh" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule source mac="00:11:22:33:44:55" port port="22222" protocol="tcp" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source mac="00:11:22:33:44:55" port port="22222" protocol="tcp" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source ipset="leochan" port port="22222" protocol="tcp" accept' --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source ipset="leochan" port port="22222" protocol="tcp" reject' --permanent
```

## 结束语

富规则里面的accept可以不填写，不填写的情况下默认通过。reject可以替换成drop，区别在于reject规范，可以便于诊断和调试网络/防火墙所产生的问题。ipset可以配合定时任务添加或者删除黑名单的IP、MAC等。