---
title: 启用Google Authenticator两步验证登录Linux SSH
date: 2018-11-02 10:48:47
tags: 
  - SSH安全
  - 二步验证
categories:
  - 技术
url: login-ssh-with-2fa-authentication-by-google-authencator
---

> Linux的SSH是系统的大门，安全重要性可想而知。下面我以Linux的发行版Centos7为栗子，介绍一下如何启用Google
Authenticator作为两步验证的工具保护我们的SSH。

<!--more-->

## 安装ntpd

```
yum install -y ntpdate
systemctl start ntpd
systemctl enable ntpd
```


ntpdate 是用来自动同步时间的程序，这里启动它并设置它开机自动启动。

## 安装必需的组件

```
yum install -y git make gcc libtool pam-devel
```

## 安装Google Authenticator

```
git clone https://github.com/google/google-authenticator
cd google-authenticator/libpam
./bootstrap.sh
./configure
make
make install
ln -s /usr/local/lib/security/pam_google_authenticator.so /usr/lib64/security/
```

安装epel-release的前提下，可利用yum install google-authenticator快速安装

## 配置 SSH 服务

编辑 /etc/ssh/sshd_config 文件

```
vim /etc/ssh/sshd_config
```

修改下面字段的配置

```
ChallengeResponseAuthentication yes
PasswordAuthentication no
PubkeyAuthentication yes
UsePAM yes
```

然后重启一下 sshd 服务，使配置生效

```
systemctl restart sshd
```


这里将 PubkeyAuthentication 配置成了 yes 表示支持公钥验证登录，即使某个账号启用了 Google Authenticator
验证，只要登录者机器的公钥在这个账号的授权下，就可以不输入密码和 Google Authenticator 的认证码直接登录。

## 配置 PAM

打开 /etc/pam.d/sshd 文件

```
vim /etc/pam.d/sshd
```

在文件顶部加入下列代码

```
auth required pam_google_authenticator.so nullok
```


## 启用 Google Authenticator

切换至想要使用 Google Authenticator 来做登录验证的账号，执行下面操作

```
google-authenticator
```

然后一路Y，再扫码就好！！登录验证的时候，只保留interactive和password，先输入手机Google
Authenticator的验证码，再输入密码，就可以登录



