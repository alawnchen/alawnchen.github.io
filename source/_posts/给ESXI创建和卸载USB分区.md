---
title: 给ESXI创建和卸载USB分区
date: 2018-11-07 07:13:02
tags: 
  - esxi
  - 分区
categories:
  - 技术
url: create-and-remove-use-partition-on-esxi
description: 首先呢，简单地说一下ESXI。ESXI是一款服务器级别的虚拟机，多被用于服务器领域，而最近火起来的很多软路由多系统共存的解决方案也都大多选择了ESXI。而我，也过了一回软路由的瘾。
---

> 首先呢，简单地说一下ESXI。ESXI是一款服务器级别的虚拟机，多被用于服务器领域，而最近火起来的很多软路由多系统共存的解决方案也都大多选择了ESXI。而我，也过了一回软路由的瘾。

## 故事的开头

把软路由的硬件处理好，安装好爱快+Lede之后，我还想着试点其他的小东西，尴尬的是存储空间不够了。我实在不想因为一个临时性的想法去把SSD升级一下，毕竟我不打算用来存储内容。巧了，家里面有几个U盘闲置着，所以，我动了点歪脑筋。

## SSH开启

ESXI默认并不启动SSH服务，所以，我们需要手动启动一下。打开ESXI的Web
UI管理界面，我们在左侧选择“主机”-“管理”，在右侧选择服务，我们在列出来的服务里面直接把“TSM”和“TSM-Shell”开启。值得注意的一个小细节是，用SSH客户端连接的时候会提示使用私钥文件登录，我们选择键盘交互，回车一下即可登录。

## 创建存储

首先，我们需要把USB控制器服务关闭。

```
/etc/init.d/usbarbitrator stop
```

然后把USB控制器服务的自启动禁用，当然，这个是一个可选操作。

```
chkconfig usbarbitrator off
```

接下来，把U盘插入，获取U盘的标识符（类似mpx.vmhbaXX格式）。

```
ls /dev/disks/
```

写入一个GPT标签（我的U盘标识符是mpx.vmhba33）

```
partedUtil mklabel /dev/disks/mpx.vmhba33\:C0\:T0\:L0 gpt
```

为了创建一个分区，需要知道的数据有USB设备的起始扇区、结束扇区、USB容量大小，以及确定一个GUID

```
partedUtil getptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L0
```

执行以上命令之后会得到USB的结束扇区的位置，我们还可以通过以下命令直接获取到正确的结束扇区

```
eval expr $(partedUtil getptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L0 | tail -1 | awk '{print $1 " \\* " $2 " \\* " $3}') - 1
```

创建分区，记得替换掉结束分区的位置

```
partedUtil setptbl /dev/disks/mpx.vmhba33\:C0\:T0\:L0 gpt "1 2048 31278554 AA31E02A400F11DB9590000C2911D1B8 0"
```

通过VMFS5格式化分区

```
vmkfstools -C vmfs5 -S USB-STORE /dev/disks/mpx.vmhba33\:C0\:T0\:L0:1
```

需要注意的是，格式化需要一定时间，期间最好耐心等待，不要去马上去操作出现在ESXI的USB-STORE分。格式化完成后，控制台会给出相应的提示。

## 卸载USB分区

这个就简单很多了，首先确保在分区里面的虚拟机处于关机状态，然后把虚拟机全部删除后，直接把分区删除掉，最后，恢复一下USB控制器服务的自启动即可。

```
chkconfig usbarbitrator on
```


启动一下USB控制器服务

```
/etc/init.d/usbarbitrator start
```