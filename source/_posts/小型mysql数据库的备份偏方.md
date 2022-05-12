---
title: 小型mysql数据库的备份偏方
date: 2018-07-12 07:57:00
tags: 
  - mysql
  - 数据库备份
categories:
  - 技术
url: a-backup-method-for-mini-mysql-database
description: 有时候，自己搞一个博客、小型论坛再或者一个小型的项目，如果又不想选择xtrabackup或者自己写个小程序备份的话，这里有一个很不错的小方案可以选择哟！
---

> 有时候，自己搞一个博客、小型论坛再或者一个小型的项目，如果又不想选择xtrabackup或者自己写个小程序备份的话，这里有一个很不错的小方案可以选择哟！

## 前言

我的Linux系统是centos，是最新版本的7.4，64bit，系统内核版本（可通过uname -a命令查看系统内核版本）3.10.0-693.2.2.el7.x86_64。这里会用到mysql自带的mysqldump工具。

```
shell脚本内容
#!/bin/sh
#mysql账户
mysql_user="leo"
#mysql密码
mysql_password="leo123456789"
#mysql自带的可执行文件目录
mysql_dir=/usr/local/mysql/bin
#要导出的数据库名称，多个用半角逗号隔开
mysql_databases="leo"
#备份目录，一定要事先创建好
backup_dir=/data/backup/mysql/leo
#导出数据库文件前缀
today=$(date +%Y%m%d_%H%M%S)
#日志目录 也要提前创建
log_dir=/data/logs/mysql/leo
#日志名称
log_file=backup.log
#追加开始备份的提示文字到日志文件
echo -e '['$(date +"%Y-%m-%d %H:%M:%S")'] - '$mysql_databases' - '"备份开始\n" >> $log_dir/$log_file
#使用mysqldump导出数据
$mysql_dir/mysqldump -u$mysql_user -p$mysql_password --apply-slave-statements --hex-blob --routines --single-transaction --databases $mysql_databases | gzip > $backup_dir/$today.sql.gz
#追加删除旧备份数据的提示文字到日志
echo -e '['$(date +"%Y-%m-%d %H:%M:%S")'] - '$mysql_databases' - '"删除以前的备份\n" >> $log_dir/$log_file
#保留多少天内的备份
days=3
#查找应该删除的数据 并删除
find $full_backup_dir -mtime +$days -name "*.sql.gz" -exec rm -f {} \;
#休息一下
sleep 600
```

我们把上面的脚本内容保存成backup.sh文件，保存在/data目录（即，文件地址是/data/backup.sh）。注意转换编码，不然会报错，提示找不到/bin/sh，最好在Linux或者类Unix系统下面保存成文件。

## 定时执行脚本

```
crontab -e
```

执行上面的命令，进入编辑定时任务模式。按一下“i”，把下面的内容放到新的一行。每6小时执行一次。

```
1 */6 * * * /data/backup.sh >> /dev/null
```

按一下“Esc”键，输入:wq!，按一下回车键保存。

如果害怕搞错，可以直接使用echo命令追加内容到定时任务文件里面！

```
echo '1 */6 * * * /data/backup.sh >> /dev/null' >> /var/spool/cron/root
```


centos系统的定时任务文件对应的目录是 /var/spool/cron，文件以用户名命名！

## 拓展一下

如果觉得还是不够6，可以参考一下《[利用inotify+rsync实时同步数据到远程服务器之二](/sync-data-realtime-with-inotify-rsync-v2)》，把备份出来的数据实时同步到其他服务器上面！不费力，也很稳定，小型数据库，足矣！