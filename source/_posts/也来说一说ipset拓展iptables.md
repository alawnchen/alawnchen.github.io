---
title: 也来说一说ipset拓展iptables
date: 2018-01-13 07:10:00
tags: 
  - ipset
  - iptables
  - 防火墙
categories:
  - 技术
url: talking-about-iptables-and-ipset
---

> ipset是一个很好的东西，它为大量地封禁IP提供了一个非常好的解决方案。初次使用ipset拓展iptables也遇上了不少问题，几经折腾，也终于得到了完美的解决！

<!--more-->


## 安装iptables

具体的安装方法，网上有很多，也可以参考我自己记录的《[在centos7上安装iptables作为防火墙](/install-iptables-as-firewall-on-centos7)》。大家最好字字句句看清楚，有一些注释说明很重要。

## 安装ipset

```
yum -y install ipset
```


## 设置开机启动

```
wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/home:/mludvig:/elml/CentOS_CentOS-6/noarch/ipset-init-1.0-10.1.noarch.rpm
rpm -ivh ipset-init-1.0-10.1.noarch.rpm
```

这个软件包叫做ipset-init。之所以要安装这个包，是因为我用的是centos6.5，ipset提供的init.d脚本好像有问题！ipset-init包内容如下：

```
/etc/rc.d/init.d/ipset
/etc/sysconfig/ipset
/etc/sysconfig/ipset-config
```

使用chkconfig命令添加开机启动


```
chkconfig --add ipset
```

## 配置ipset

接下来的工作主要是创建一个ipset集合，并保存它！！

```
ipset create babe hash:ip hashsize 1024 maxelem 1000000 timeout 86400
service ipset save
```

babe是集合名称。hash是指定格式用的,，更多格式可以通过ipset
--help查看。hashsize是指初始化条数空间，maxelem是指最大条数空间。timeout指的并不是集合的过期时间，而是新增hash内容的默认过期时间。如果不需要超时，可以吧timeout参数去掉即可。

添加IP到ipset集合里面

```
ipset add babe 1.1.1.1 timeout 3600
```

我们需要注意点的是，如果创建集合的时候没有使用timeout参数，那么我们添加IP的时候，也不能使用timeout，如下：

```
ipset add babe 1.1.1.1
```

添加完了，得保存一下，不然重启会丢失

```
service ipset save
```

自动处理添加ipset集合hash内容
假设有一个文件是/data/some/block.list，里面一行有一个IP，我们需要将这些IP都加入ipset的babe集合里面。
首先，创建一个叫做block.sh的脚步，它的路径是/data/some/block.sh。脚本内容如下：

```
grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' /data/some/block.list | awk '{print "ipset add bklist",$0}'|sh
service ipset save
service iptables reload
```

接下来，我们让系统每半小时执行一次上面的脚本。

```
crontab -e
0,30 */1 * * * /data/some/block.sh > /dev/null 2>&1
```

保存之后即可。

## 配置iptables

将集合作为黑名单

```
iptables -I INPUT -m set --match-set babe src -j DROP
```

将集合作为白名单

```
iptables -I INPUT -m set ! --match-set babe src -j DROP
```

执行完了，需要保存并重载防火墙内容。

```
service iptables save
service iptables reload
```