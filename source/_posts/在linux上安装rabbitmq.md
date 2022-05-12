---
title: 在linux上安装rabbitmq
date: 2018-08-17 08:00:00
tags: 
  - rabbitmq
  - 队列
categories:
  - 技术
url: install-rabbitmq-on-linux
description: 最近，在一个项目里面用到了RabbitMQ。消息队列用过不少，不过，RabbitMQ还真的是第一次接触。第一次也碰上了一个很有意思的小地方。
---

最近，在一个项目里面用到了RabbitMQ。消息队列用过不少，不过，RabbitMQ还真的是第一次接触。第一次也碰上了一个很有意思的小地方。

## 环境说明

服务器系统是centos7.4，64bit，系统内核版本（可通过uname -a命令查看系统内核版本）3.10.0-693.2.2.el7.x86_64。此外，因为最后是安装了对应的PHP
C扩展，所以也补充一下PHP的版本，是7.1.19。

## 安装RabbitMQ服务端

RabbitMQ服务端的安装就比较简单了，只需要通过yum包管理器安装即可。安装完成之后，启动RabbitMQ.

```
yum install rabbitmq-server -y
systemctl start rabbitmq-server
```

如果提示找不到RabbitMQ包，可先安装EPEL源之后，再执行上面的步骤。

```
yum install epel-release -y
```

EPEL源虽然不是官方维护的源，但是却也是一个第三方里最大的一个源。它不仅提供了大量的RPM包，且绝大多数RPM包要比官方源的RPM包版本更新，是一个RHEL
及衍生发行版如 CentOS、Scientific Linux最可靠的一个源！

## 安装rabbitmq-c

RabbitMQ对应的PHP C扩展amqp是基于C客户端rabbitmq-c开发的，所以，安装amqp之前，我们必须把rabbitmq-c安装上。当前最新版本是v0.9.0。

```
wget https://github.com/alanxz/rabbitmq-c/archive/v0.9.0.tar.gz
tar xzf v0.9.0.tar.gz
cd rabbitmq-c*
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/rabbitmq-c ..
cmake --build .  --target install
ln -s /usr/local/rabbitmq-c/lib64 /usr/local/rabbitmq-c/lib
```

cmake是一个很好用的一个编译工具，它最大的特点就是无关平台，即跨平台性。上面的cmake命令作用在于生成Makefile。-DCMAKE_INSTALL_PREFIX，可以拆分看成-D和CMAKE_INSTALL_PREFIX。CMAKE_INSTALL_PREFIX是一个变量，类似于在使用configure脚本时候用的PREFIX。

至于给lib64创建软连接，是因为在编译安装amqp时候，系统会到/usr/local/rabbitmq-c/lib扫描，查找依赖库，而不巧的是压根没有/usr/local/rabbitmq-c/lib目录。

## 安装amqp

```
wget https://pecl.php.net/get/amqp-1.9.3.tgz
cd amqp-1.9.3
phpize
./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp --with-librabbitmq-dir=/usr/local/rabbitmq-c
make
make install
```

## 加载amqp

```
echo 'extension=amqp.so' >> /usr/local/php/etc/php.ini
service php-fpm restart
```

## 最后验证一下amqp正常加载

```
php -m
```

发现有amqp，那就OK了！