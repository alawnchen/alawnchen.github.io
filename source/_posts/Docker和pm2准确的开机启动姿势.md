---
title: Docker和pm2准确的开机启动姿势
date: 2018-10-28 20:08:13
tags: 
  - docker
  - pm2
  - 开机自启
categories:
  - 技术
url: the-right-way-of-startup-to-docker-and-pm2
description: Docker和pm2是不同的东西，不过都同为新生技术产物。所以，很多东西都还需要时间去摸索和熟悉。想想，Docker和pm2的开机自启就曾经困扰了很多人吧。下面说一下这两个东西的开机自启功能吧！
---

> Docker和pm2是不同的东西，不过都同为新生技术产物。所以，很多东西都还需要时间去摸索和熟悉。想想，Docker和pm2的开机自启就曾经困扰了很多人吧。下面说一下这两个东西的开机自启功能吧！

## Docker开机自启

很多人说要写docker开机启动脚本啊，要不在init.d里面添加啊，又或者追加到rc.local里面去。其实远没有那么麻烦，因为docker有着自己的方式！

```
docker run --restart=always ……
```

## pm2开机自启

pm2的应用，大家都基本知道的。作为nodejs的一个进程管理利器，pm2绝对是足够的强悍了！说到pm2的随机启动，还是会有很多人不清楚。不过，一样的，pm2也有自己的启动方式！

```
pm2 start /home/node/web/pm2.json
pm2 save
pm2 startup
pm2 stop all
pm2 kill
```

做了上面的步骤之后呢，系统的**/etc/systemd/system**里面会多出一个pm2.service文件。这个自然不必多说，想必大家都知道是干嘛用的。下面我们让pm2.service应用生效即可

```
systemctl enable pm2.service
```


## 注意

上面的例子都是以centos7为参照的，centos的其他版本、其他的linux发行版本或者macOS可能有所不同。大家可以多多尝试！

