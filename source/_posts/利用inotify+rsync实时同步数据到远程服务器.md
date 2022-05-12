---
title: 利用inotify+rsync实时同步数据到远程服务器
date: 2018-10-29 16:48:47
tags: 
  - inotify
  - rsync
  - 实时同步
categories:
  - 技术
url: sync-data-realtime-with-inotify-rsync
---

> 数据的价值和重要性历来都很重要。大数据时代，更是如此。而备份数据是最常用也是最有效的保护数据的方式。而我利用inotify+rsync达到了实时同步数据从而备份数据的目的！！

<!--more-->

被备份的服务器简称备A，IP是192.168.1.1；
备份数据存放的服务器简称远B，IP是192.168.1.2
两个服务器的系统均为Centos 7.3 64bit

## 安装rsync

```
rpm -qa|grep rsync #检查是否安装过rsync，whereis rsync也可以
yum install rsync -y #如果未安装，使用yum安装rsync
```


备A和远B都必须安装rsync，而且版本最好一样！！Centos一般都默认自带rsync，可利用rsync --version或者rsync -h查看版本。

## 在备A上安装inotify

```
wget http://cloud.github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz  
tar zxvf inotify-tools-3.14.tar.gz  
cd inotify-tools-3.14  
./configure --prefix=/usr/local/inotify  
make  
make install
```


## 配置备A的rsync

```
mkdir -p /usr/local/rsync
touch /usr/local/rsync/exclude.list
echo 'testpassword' >/usr/local/rsync/rsync.passwd
chmod 600 /usr/local/rsync/rsync.passwd
```


exclude.list是用来排除文件和目录，先暂时留空；rsync.passwd是密码文件，只存放密码。密码文件必须给读写权限，否则会报错

## 配置远B的rsync

```
mkdir -p /usr/local/rsync
echo 'user:testpassword' >/usr/local/rsync/rsync.passwd
chmod 600 /usr/local/rsync/rsync.passwd
touch /usr/local/rsync/rsync.conf
```

rsync.passwd是用来存放用户名和密码的，user和testpassword分别是用户名和密码，testpassword必须和备A的rsync.passwd里面的密码一样！！rsync.conf是配置文件，内容设置如下：

```
pid file = /var/run/rsyncd.pid  
lock file = /var/run/rsync.lock  
log file = /var/log/rsyncd.log 
port = 873
address = 192.168.1.2
secrets file=/usr/local/rsync/rsync.passwd
uid = root
gid = root
use chroot = yes
read only = no
write only = no
#允许访问rsyncd服务的ip，ip端或者单独ip之间使用空格隔开
hosts allow = 192.168.1.1
#不允许访问rsyncd服务的ip，*是全部(不涵盖在hosts allow中声明的ip，注意和hosts allow的先后顺序)
hosts deny = *
#客户端最大连接数
max connections = 5
#日志相关
#    log file 指定rsync发送消息日志文件，而不是发送给syslog，如果不填这个参数默认发送给syslog
#    transfer logging 是否记录传输文件日志
#    log format 日志文件格式，格式参数请google
#    syslog facility rsync发送消息给syslog时的消息级别，
#    timeout连接超时时间
log file = /usr/local/logs/rsync.log
transfer logging = yes
log format = %t %a %m %f %b
syslog facility = local3
timeout = 300
[leo]
#模块根目录，必须指定 用于存放备份数据，可以任意指定，文件夹是在远B服务器上
path=/data/leo/
#是否允许列出模块里的内容
list=yes
#忽略错误
ignore errors
#模块验证用户名称，可使用空格或者逗号隔开多个用户名
auth users = user
#注释
comment = some description about this moudle
#排除目录，多个之间使用空格隔开 一般只要在备A配置排除就好
#exclude = test1/ test2/
```


配置还可以稍微修改并简化一点

```
uid = root 
gid = root 
use chroot = no 
max connections = 10 
strict modes = yes 
pid file = /var/run/rsyncd.pid  
lock file = /var/run/rsync.lock  
log file = /var/log/rsyncd.log  
[leo]  
path = /data/leo/  
comment = backup wwwroot  
ignore errors  
read only = no 
write only = no 
hosts allow = 192.168.1.1  
hosts deny = *  
list = false 
uid = root 
gid = root 
auth users = user
secrets file = /usr/local/rsync/rsync.passwd
```

## 启动远B的rsync

```
/usr/bin/rsync --daemon --config=/usr/local/rsync/rsync.conf
```


为了方便，可以让远B的rsync随机启动，我们需要将启动命令加入/etc/rc.local文件

```
echo '/usr/bin/rsync --daemon --config=/usr/local/rsync/rsync.conf' >> /etc/rc.local
chmod +x /etc/rc.local
```

启动脚本如果没有执行权限，那么随机启动会失败，所以运行了chmod +x /etc/rc.local。启动后，可以通过netstat -an | grep 873 检验是否已经启动rsync

## 开放远B的rsync端口

```
iptables -I INPUT 3 -p tcp -m state --state NEW  -m tcp --dport 873 -j ACCEPT
service iptables save
service iptables reload
```


## 在备A创建备份脚本

假设，我们需要备份的是/data/leo文件夹，我们需要排除/data/leo/root文件夹和/data/leo/sh.sh文件
我们在/data/leo创建rsync.sh，并填充如下内容：

```
#!/bin/bash
host=192.168.1.2
src=/data/leo/
des=leo
user=user
/usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src | while read files
do
/usr/bin/rsync -vzrtopg --delete --progress --exclude-from=/usr/local/rsync/exclude.list --password-file=/usr/local/rsync/rsync.passwd $src $user@$host::$des
echo "${files} was rsynced" >>/data/wwwlogs/rsync.log 2>&1
done
```

host是远B的IP；src是备A服务器上的备份源文件夹；des是远B配置文件/usr/local/rsync/rsync.conf里面的[leo]的leo，也就是说这个模块名称由远B的配置文件决定。如果备A删除文件，而要求远B保留被删除文件，则将--delete参数去除

排除/data/leo/root文件夹和/data/leo/sh.sh

```
echo 'root/' >> /usr/local/rsync/exclude.list
echo 'sh.sh' >>/usr/local/rsync/exclude.list
```

一行一个排除规则，只能用相对路径

## 开机启动备A的启动脚本

```
echo 'bash /data/leo/rsync.sh &' >> /etc/rc.local
chmod +x /etc/rc.local
chmod 744 /data/leo/rsync.sh
```


## 一些错误

1.rsync: failed to connect to X.X.X.X: No route to host
远B没有开放873端口的原因，备A无法访问远B

2.rsync failed Connection refused 111
远B的rsync没有启动，启动即可