---
title: 解除denyhosts黑名单里面的IP
date: 2018-01-13 07:07:00
tags: 
  - denyhosts
categories:
  - 技术
url: remove-the-ip-blocked-by-denyhosts
---

> denyhosts为我们保护服务器提供了一种很好的保护机制。但是有时候使用不当，自己的IP就会被加入到黑名单里面！！我现在以Centos7.3为例，说说怎么解除被封禁的IP吧。其他版本的Linux发行版的方式，大家可以举一反三！！

<!--more-->

## 方法一

```
echo '' > /usr/share/denyhosts/data/hosts
echo '' > /usr/share/denyhosts/data/hosts-restricted
echo '' > /usr/share/denyhosts/data/hosts-root
echo '' > /usr/share/denyhosts/data/hosts-valid
echo '' > /usr/share/denyhosts/data/users-hosts
echo '' > /etc/hosts.deny
echo '' > /var/log/secure
```


上面的shell命令主要是为了覆盖上面的几个文件。当然我们可以使用tee命令结合echo批量处理。

```
echo '' | tee /usr/share/denyhosts/data/hosts /usr/share/denyhosts/data/hosts-restricted /usr/share/denyhosts/data/hosts-root /usr/share/denyhosts/data/hosts-valid /usr/share/denyhosts/data/users-hosts /etc/hosts.deny /var/log/secure
```

这个方法很粗暴，不保留任何的失败记录和黑名单。如果是单个IP或者是不介意丢失那些信息，那这个是最好最便捷的方法。如果介意，那可以试试方法二。

## 方法二

```
sed -i '/192.168.1.1/d' /usr/share/denyhosts/data/hosts
sed -i '/192.168.1.1/d' /usr/share/denyhosts/data/hosts-restricted 
sed -i '/192.168.1.1/d' /usr/share/denyhosts/data/hosts-root
sed -i '/192.168.1.1/d' /usr/share/denyhosts/data/hosts-valid
sed -i '/192.168.1.1/d' /usr/share/denyhosts/data/users-hosts
sed -i '/192.168.1.1/d' /etc/hosts.deny
sed -i '/192.168.1.1/d' /var/log/secure
```

上面的shell命令是在那些文件里面删除含有192.168.1.1的行！！