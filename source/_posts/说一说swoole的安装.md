---
title: 说一说swoole的安装
date: 2018-08-30 08:07:00
tags: 
  - swoole
categories:
  - 技术
url: about-swoole-installtion
description: 因为项目需求，接触过几次swoole。swoole把项目的实现难度降低了一个维度，是一个实实在在的好东西。不过，这个东西也很让很多新手头疼吧，毕竟是一个基于C开发出来的PHP扩展。我也为swoole普及使点力，就说一说swoole的编译安装吧。
---

> 因为项目需求，接触过几次swoole。swoole把项目的实现难度降低了一个维度，是一个实实在在的好东西。不过，这个东西也很让很多新手头疼吧，毕竟是一个基于C开发出来的PHP扩展。我也为swoole普及使点力，就说一说swoole的编译安装吧。

## 环境说明

服务器系统是centos7.4，64bit，系统内核版本（可通过uname
-a命令查看系统内核版本）3.10.0-693.2.2.el7.x86_64。此外，因为最后是安装了对应的PHP
C扩展，所以也补充一下PHP的版本，是7.1.19。安装的swoole是最新的v4.0.4，支持异步Redis客户端以及HTTP2。这里的说明不涉及Redis的编译安装，因为这个很常用，只需要确保安装的redis版本一定要大于或者等于1.2，原因后面说。

## 安装hiredis

swoole异步redis客户端的实现依赖于hiredis。hiredis是一个基于C开发的redis客户端，要求redis版本必须等于或者大于1.2。github地址是<https://github.com/redis/hiredis>

```
wget https://github.com/redis/hiredis/archive/v0.13.3.tar.gz
tar xzf v0.13.3.tar.gz
cd hiredis*
make
sudo make install
```

## 安装nghttp2

安装nghttp2是为了解决swoole支持http2的依赖。github地址是<https://github.com/nghttp2/nghttp2>

```
wget https://github.com/nghttp2/nghttp2/releases/download/v1.32.0/nghttp2-1.32.0.tar.bz2
tar -jxvf nghttp2-1.32.0.tar.bz2
cd nghttp2*
./configure
make
sudo make install
```

## 安装swoole

这里说一下swoole的版本，有1.×，2.x以及4.x。这里做一下简单的说明，1.x没有支持协程，2.x和4.x都支持协程，所不同者在于4.x是基于boost.context汇编代码实现了全新的协程内核，两者之间没有兼容问题。值得注意的是，1.x和2.x都不会再开发新的特性，所以，如果想多尝试，可选择4.x。

```
wget https://github.com/swoole/swoole-src/archive/v4.0.4.tar.gz
tar xzf v4.0.4.tar.gz
cd swoole*
phpize
./configure --enable-sockets --enable-openssl --enable-http2 --enable-async-redis --enable-mysqlnd
make
sudo make install
```

## 加载swoole

```
echo 'extension=swoole.so' >> /usr/local/php/etc/php.ini
service php-fpm restart
```

在终端输入php -m | grep swoole，有结果就行！

## 后话

如果重载php fpm的时候出现类似的“libhiredis.so.0.13: cannot open shared object
file”的错误，是因为swoole找不到动态库的原因。解决的办法有两个：
编译安装的swoole的时候修改参数

```
./configure --enable-sockets --enable-openssl --enable-http2 --enable-async-redis=/usr/local --enable-mysqlnd
```


或者修改ld.so.conf，让系统搜索动态链接库的时候也搜索指定目录。

```
echo '/usr/local/lib' >> /etc/ld.so.conf
sudo ldconfig
```

重载php-fpm即可