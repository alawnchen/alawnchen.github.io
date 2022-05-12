---
title: 利用inotify+rsync实时同步数据到远程服务器之二
date: 2018-06-22 07:47:00
tags: 
  - inotify
  - 实时同步
categories:
  - 技术
url: sync-data-realtime-with-inotify-rsync-v2
description: inotify的新版本出来许久，我才发现，现在把它升级一下吧。目下，inotify的新版本号是3.20.1。
---

> inotify的新版本出来许久，我才发现，现在把它升级一下吧。目下，inotify的新版本号是3.20.1。它的Github地址是<https://github.com/rvoicilas/inotify-tools>

最近因为备份数据，在服务器上面配置inotify+rsync实时同步（参考《[利用inotify+rsync实时同步数据到远程服务器](/sync-data-realtime-with-inotify-rsync)》）时发现inotify升级了。于是打算把inotify升级一下，顺便把原来用/etc/rc.local开机启动的方式改成用supervisor守护进程。

## 安装inotify

对比以前的inotify老版本，比较新的版本的源码包里面都没有configure这个文件，所以呢，安装的时候需要生成configure可执行文件。参考原来的文章，发现以前的下载链接已经没有了，所以，前后文件变化，我也懒得去比较了。不过，确定的一点是，在inotify迁移到github上面之后的所有版本都没有configure。

新版本源码里面有autogen.sh，是用来自动构造一些配置和可执行文件的shell脚本。而需要的configure文件也是通过它构造的！

```
wget https://github.com/rvoicilas/inotify-tools/archive/3.20.1.tar.gz  
tar zxvf 3.20.1.tar.gz 
cd inotify-tools-3.20.1 
./autogen.sh
./configure --prefix=/usr/local/inotify  
make  
make install 
```

和原来差不多，多了一步执行autogen.sh

## supervisor守护远程服务器B的rsync

```
[program:rsync]
command=/usr/bin/rsync --daemon --config=/usr/local/rsync/rsync.conf
numprocs=1
autostart=true
autorestart=true
startretries=10
exitcodes=0
stopsignal=KILL
stopwaitsecs=10
redirect_stderr=true
stdout_logfile=logfile
user=root
redirect_stderr=true
stdout_logfile=/data/supervisor/log/rsync.log
loglevel=error
```


这里守护的远程服务器B的rsync（参考原来文章），因为需要确保远程服务器的rsync处于常开状态！

## supervisor守护被备份服务器A的服务

```
[program:rsync]
command=bash /data/leo/rsync.sh
numprocs=1
autostart=true
autorestart=true
startretries=10
exitcodes=0
stopsignal=KILL
stopwaitsecs=10
redirect_stderr=true
stdout_logfile=logfile
user=root
redirect_stderr=true
stdout_logfile=/data/supervisor/log/rsync.log
loglevel=error
stdout_logfile_maxbytes=5MB
stdout_logfile_backups=3
```

被备份服务器的情况是，利用inotify侦听目录变化，然后循环调用rsync同步数据到远程服务器B。

## 最后

supervisor守护进程的说明，可参考《[Linux上用supervisor守护进程](/run-daemon-process-with-supervisor-on-linux)》。