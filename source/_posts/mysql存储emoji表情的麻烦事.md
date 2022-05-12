---
title: mysql存储emoji表情的麻烦事
date: 2018-05-10 07:45:00
tags: 
  - mysql
  - emoji
categories:
  - 技术
url: storing-emoji-in-mysql
---

> 最近都在和微信相关的平台打交道，微信的有些用户用emoji表情作为昵称的一部分或者全部，那真的是很可爱。可是就是看起来很可爱的emoji，处理起来是一点都不可爱！

<!--more-->

## emoji的由来

emoji这玩意，“很不幸”地被日本发明，最初是日本在无线通信中所使用的视觉情感符号。作为国际范，emoji已经像空气一样充满整个世界，它在中国有一个贴切的中文名叫做“小黄脸”。这玩意无处不在了，支持它是不存在争议的！

## emoji的编码

emoji表情对应的Unicode编码在mysql数据库里面，需要用到的字符集是utf8mb4。unicode.org
专门整理了一个完整的emoji的编码表。具体地址：https://unicode.org/emoji/charts/full-emoji-list.html

## 存储emoji

回到正题，如果要在mysql数据库里面存储emoji，就需要用到utf8mb4字符集。而且注意的是，必须是数据库默认字符集、数据表默认字符集以及存储它的具体字段字符集也是utf8mb4。

```
mysql -u root -p
```

输入正确的密码，进入mysql命令行模式。

```
mysql> use leo;
mysql> alter database leo default charset = utf8mb4;
mysql> alter table leo_table default charset = utf8mb4;
mysql> alter table leo_table change table_emoji table_emoji VARCHAR(128) character set utf8mb4 collate utf8mb4_general_ci;
```

上面的leo是数据库名称，leo_table是数据表，table_emoji是数据库字段。（mysql>只是为了表示是mysql命令行模式下执行，无作用）。依次作用如下：

1. 切换数据库;
2. 更改数据库字符集；
3. 更改数据表字符集；
4. 更改字段字符集；

需要特别注意的是最后的更改字符集，一定要确定字段的类型、长度等定义。

## 修改mysql配置文件my.cnf

```
[client]  
default-character-set = utf8mb4  
  
[mysql]  
default-character-set = utf8mb4  
  
[mysqld]  
character-set-client-handshake = FALSE  
character-set-server = utf8mb4  
collation-server = utf8mb4_unicode_ci  
init_connect='SET NAMES utf8mb4' 
```

注意对比，没有的就新增，存在的修改即可！

## 最后

补充一下，字符集utf8mb4是在mysql5.5.3以上版本后才出现的，所以要确保mysql版本不能太低。重启一下mysql服务，就可以愉快地存储这可爱的emoji表情了。