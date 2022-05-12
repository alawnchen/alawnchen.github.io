---
title: nginx+lua编译那点事情
date: 2018-02-01 07:19:00
tags: 
  - nginx
  - lua
  - 防火墙
categories:
  - 技术
url: install-nginx-and-lua
description: nginx嘛，自不必说了，最好使的服务器软件之一。lua的特点，想必很多人也是心知肚明的。而OpenResty给我们看到了一种nginx的新玩法，那就是nginx+lua。
---

> nginx嘛，自不必说了，最好使的服务器软件之一。lua的特点，想必很多人也是心知肚明的。而OpenResty给我们看到了一种nginx的新玩法，那就是nginx+lua。

## 准备

首先，我们需要下载nginx1.12.0、luajit2.0.5以及lua-nginx-modulev0.10.9rc8，并把他们都存放在/root/src目录里面。他们在github上面的网址分别是https://github.com/nginx/nginx  、https://github.com/LuaJIT/LuaJIT  以及 
https://github.com/openresty/lua-nginx-module

```
mkdir -p /root/src
cd /root/src
wget https://github.com/nginx/nginx/archive/release-1.12.0.tar.gz
wget https://github.com/LuaJIT/LuaJIT/archive/v2.0.5.tar.gz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc8.tar.gz
```

## 全部解压他们

```
ls *.tar.gz | xargs -n1 tar xzvf
```

## 安装luajit

```
cd LuaJIT-2.0.5
make
make install PREFIX=/usr/local/luajit
```

## 编译安装nginx

```
cd nginx-1.12.0
export LUAJIT_LIB=/usr/local/luajit/lib
export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0
./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_stub_status_module --with-http_v2_module --with-http_ssl_module --with-http_gzip_static_module --with-http_realip_module --add-module=/root/src/lua-nginx-module-0.10.9rc8
make
make install
```

## 错误

编译安装nginx后，启动nginx的时候，可能会出现类似libluajit-5.1.so.2: cannot open shared object file的错误提示。那是因为系统找不到libluajit-5.1.so.2的原因，建立个软连接即可。

```
ln -s /usr/local/luajit/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
```