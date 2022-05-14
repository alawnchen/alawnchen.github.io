---
title: 定时开关OpenWRT的LED
date: 2020-03-03 23:46:07
tags: 
  - 软路由
  - OpenWRT
categories:
  - 技术
url: ding-shi-kai-guan-openwrtde-led
description: 家里面，不管是老家用的路由器，还是在美食之都用的软路由，用的都是OpenWRT系统。用上OpenWRT之后，抖音不卡，微信666。不过，它的指示灯实在是太亮了，尤其是夜里面！所以呢，就折腾了一下，定时开关OpenWRT的LED。
---

> 家里面，不管是老家用的路由器，还是在美食之都用的软路由，用的都是OpenWRT系统。用上OpenWRT之后，抖音不卡，微信666。不过，它的指示灯实在是太亮了，尤其是夜里面！所以呢，就折腾了一下，定时开关OpenWRT的LED。
 
## 确定配置文件所在

首先，我通过Google搜索，查到了[OpenWRT官网LED的说明文档](https://openwrt.org/docs/guide-user/base-system/led_configuration)。查到了相关的配置文件夹以及基本工作原理。经过比对文档和文件，发现我们可以控制触发器（trigger）和亮度（brightness）来达到控制LED。在这里，只通过控制亮度去控制LED，简单高效。

## 任务脚本

首先，我们需要做简单的准备。需要确定的是，我们需要一个文件夹备份原来的亮度配置文件。关掉灯就是把亮度变成0即可，开灯就把原来的配置恢复即可。

```
mkdir -p /data/leds #创建一个基本文件夹
touch switch.sh #创建脚本文件
chmod a+x switch.sh
```


编辑并保存一下内容到switch.sh

```
#!/bin/bash
#
# author: Leo Chan
# Blog: https://leochan.me
#

backPath="/data/leds/backup/"
ledsPath="/sys/class/leds/"

backupConfig(){
	if [ ! -d $backPath ]; then
		mkdir -p $backPath
		for i in `find / -name brightness`; do
			name=`dirname $i | grep -o '[^/]*$'`
			cat $i > $backPath$name
		done
	fi
}

turnOff(){
	for i in `ls "$backPath"`
	do
	  	echo 0 > "$ledsPath$i/brightness"
	done
}

turnOn(){
	for i in `ls "$backPath"`
	do
	  	cat $backPath$i > "$ledsPath$i/brightness"
	done
}

switchLed(){
	if [ `date +%H` -gt 19 ]; then
		turnOff
	else
		turnOn
	fi
}

backupConfig
switchLed
```


## 添加定时任务

可以通过界面添加，也可以通过crontab命令编辑添加。我自己是让它在晚上8点钟和白天八点钟各执行一次。

```
* 8 * * * /data/leds/switch.sh >/dev/null 2>&1
* 20 * * * /data/leds/switch.sh >/dev/null 2>&1
```