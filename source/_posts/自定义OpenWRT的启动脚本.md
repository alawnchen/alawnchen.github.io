---
title: 自定义OpenWRT的启动脚本
date: 2020-02-19 18:37:44
tags: 
  - OpenWRT
  - 自启动
categories:
  - 技术
url: zi-ding-yi-openwrtde-qi-dong-jiao-ben
---

> OpenWRT是最流行的开源路由器系统，貌似没有之一。很出名的Lede是基于OpenWRT开发的，甚至于最近声名鹊起的国内企业开发的高恪路由器系统也是基于OpenWRT开发的。对于使用者来说，自定义OpenWRT的启动脚本，也是一种必要的技能！！

<!--more-->

## 建立启动脚本

找一个地方创建一个shell脚本，强烈建议不要放置在/tmp目录，可能会被清除释放。例如：在/home下面新建一个脚本叫做leochan.me.sh。

```
cd /home
echo '#!/bin/sh' > leochan.me.sh
echo '/bin/echo leochan.me > /home/leochan.me.log' >> leochan.me.sh
chmod +x leochan.me.sh
```

这是一个很简单的脚本，只是为了说明，请自己根据需求写入相应的脚本内容。

## 创建procd脚本

OpenWRT是基于procd守护进程的，也就是说procd作为父级进程监控各个进程的状态。

```
cat << EOF > /etc/init.d/leochan
#!/bin/sh /etc/rc.common
USE_PROCD=1
START=95
STOP=01
start_service() {
    procd_open_instance
    procd_set_param command /bin/sh "/home/leochan.me.sh"
    procd_close_instance
}
EOF
chmod 755 /etc/init.d/leochan
```
上面只有两行命令，chmod 755 /etc/init.d/leochan是单独的一行，上面的是一行。

## 应用

激活脚本

```
/etc/init.d/leochan enable
```
#查看生效的启动脚本

```
ls -la /etc/rc.d/S*
```
启动脚本

```
/etc/init.d/leochan start
```
验证结果 

/home/leochan.me.log里面有leochan.me说明启动正常

```
cat /home/leochan.me.log
```

## 重要提示

最好不要直接进入/etc/init.d目录复制一份，然后仿制应用，否则会出现无法打开OpenWRT WebUI的情况。出现这种情况，只要登录SSH，进入/etc/init.d，删除掉那个复制创建的脚本，断电重启即可！！