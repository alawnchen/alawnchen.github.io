---
title: 在centos7上安装OpenCC以及PHP C扩展opencc4php
date: 2018-03-31 07:39:00
tags: 
  - OpenCC
categories:
  - 技术
url: install-the-php-c-extension-opencc
description: 基于一个项目的需要，需要在PHP上面实现繁体转简体汉字相互转换的需求。为了保证运行效率，基于C/C++写出来的PHP扩展当然是首选，所以我找到了OpenCC。下面就说说在centos7上面安装OpenCC的整个过程，PHP版本PHP7.1.12，编译了Zend OPcache扩展。
---

> 基于一个项目的需要，需要在PHP上面实现繁体转简体汉字相互转换的需求。为了保证运行效率，基于C/C++写出来的PHP扩展当然是首选，所以我找到了OpenCC。下面就说说在centos7上面安装OpenCC的整个过程，PHP版本PHP7.1.12，编译了Zend OPcache扩展。

## 安装EPEL

EPEL全称Extra Packages for Enterprise
Linux，老鸟们都知道安装上epel就相当于添加了一个很可靠的软件源一样，以后安装软件就省事情好多。如果已经安装可以跳过！

```
yum -y install epel-release
```

## 安装Doxygen

官方并没有说到需要Doxygen，是我第一次编译OpenCC的时候发现了报错Could NOT find Doxygen (missing:
 DOXYGEN_EXECUTABLE)，所以才知道也需要安装Doxygen。当然，安装过了，也可以跳过。

```
yum -y install doxygen
```

## 安装OpenCC

```
git clone https://github.com/BYVoid/OpenCC.git --depth 1
cd OpenCC
make
sudo make install
```

## 安装opencc4php

```
git clone https://github.com/NauxLiu/opencc4php.git --depth 1
cd opencc4php
phpize
./configure
make && sudo make install
```

## 可能的报错

在我把opencc.so添加到php.ini配置以后，重载了php-fpm，原本以为已经OK，但是还是报错了。

```
php -m
```

执行上面的命令查看安装成功没有的时候，发现报错如下，虽然只是一条警告级别的报错，却让opencc没办法工作：

```
**PHP Startup: Unable to load dynamic library
'/usr/local/php/lib/php/extensions/no-debug-non-zts-20160303/opencc.so' -
libopencc.so.2: cannot open shared object file: No such file or directory in
Unknown on line 0
**
```

其实就是说找不到libopencc.so.2文件，只要建立一个软连接就搞定了

```
ln -s /usr/lib/libopencc.so.2 /usr/lib64/libopencc.so.2
```

建立软连接后，再重载php-fpm，就可以愉快地在php里面使用opencc了