---
title: 说一说群晖版本的inotify+rsync同步
date: 2020-02-19 18:22:50
tags: 
  - inotify
  - 群晖
  - 实时同步
categories:
  - 技术
url: shuo-yi-shuo-qun-hui-ban-ben-de-inotify-rsynctong-bu
---

> 相信很多人都入手了NAS！群晖作为最知名的NAS品牌之一，用户数量当然不在少数了。今年上半年，在给公司选择NAS的时候，就选择了群晖。群晖的系统体验真的没得说，不过还是遇上了一些问题。最要紧的是要部署前端到机器上面，尝试了很多方法都没有办法去满足实时部署的需求。几经尝试，终于得到解决，在此分享一下。

<!--more-->

## 查找包管理器

inotify是最好的系统监控框架之一。可惜在群晖上面并没有找到相应的套件。而后，考虑过docker，但是考虑到机器硬件有限，也放弃了。因为群晖的DSM是基于linux开发的，所以转而想到利用包管理器安装一下。

1. 切换到root权限

```
sudo -i
```

2. 切换到系统根目录

```
cd /
```

3. 查找系统是不是有常见的软件包管理器

```
find . -type f ( -iname "yum" -o -iname "apt-get" -o -iname "ipkg" -o -iname "opkg" -o -iname "dpkg" )
```

通过上面，在系统里面找到了ipkg、opkg和dpkg三个包管理器。

## ipkg安装方式

通过ipkg方式安装一下。

1. 更新软件包列表

```
ipkg update
```

2. 查找是不是有inotify-tools

```
ipkg list | grep inotify-tools
```

3. 安装

```
ipkg install inotify-tools
```

4. 验证安装

```
which inotifywatch
```

## opkg安装方式

通过opkg方式安装。

1. 更新软件包列表

```
opkg update
```

2. 查找是不是有inotify-tools

```
opkg list | grep inotify-tools
```

3. 安装

```
opkg install inotify-tools
```

4. 验证安装

```
which inotifywatch
```

## dpkg安装方式

通过dpkg方式安装。

1. 下载软件包

```
wget http://ftp.br.debian.org/debian/pool/main/i/inotify-tools/inotify-tools_3.14-2_amd64.deb
```

2. 安装

```
dpkg -i inotify-tools_3.14-2_amd64.deb
```

3. 验证安装

```
which inotifywatch
```

## 配置inotify-tools和rsync

rsync已经默认安装在群晖的DSM系统上面，而inotify-tools和rsync相关配置可以参考一下《
利用inotify+rsync实时同步数据到远程服务器[/sync-data-realtime-with-inotify-rsync/]》，这里就不再多说。

## 一些说明

由于ipkg源极少，强烈建议通过opkg或者dpkg安装。群晖里面的侦听同步脚本需要开机启动，建议加入到群晖的任务计划里面。