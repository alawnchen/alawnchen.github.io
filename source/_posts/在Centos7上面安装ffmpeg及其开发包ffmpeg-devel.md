---
title: 在Centos7上面安装ffmpeg及其开发包ffmpeg-devel
date: 2018-04-20 07:41:00
tags: 
  - ffmpeg
categories:
  - 技术
url: install-ffpmeg-and-ffmpeg-devel-on-centos7
description: ffmpeg作为一个开源，成熟的多媒体播放、转换、压缩解决方案，是广大多媒体资源从业者的最好的一个选择。最近，新上线的一个项目也用到了ffmpeg，所以也来说说怎么在centos上面安装ffmpeg及其开发包ffmpeg-evel。
---

> ffmpeg作为一个开源，成熟的多媒体播放、转换、压缩解决方案，是广大多媒体资源从业者的最好的一个选择。最近，新上线的一个项目也用到了ffmpeg，所以也来说说怎么在centos上面安装ffmpeg及其开发包ffmpeg-evel。

## 约定

我所用的系统是Centos 7.4 64bit。至于其他的Linux发行版或者macOS，请自行变通处理，这里仅作为一个参考。

## yum安装

1. 首先确保已经安装了EPEL，如果已经安装，可跳过

```
yum -y install epel-release
```


2. 安装完EPEL之后，导入Nux Dextop仓库

```
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
```

3. 最后执行安装命令就好

```
yum -y install ffmpeg ffmpeg-devel
```


## shell自动安装

值得注意的是，这个脚本安装的是一个预编译版本的ffmpeg。

1. 下载shell脚本

```
wget https://raw.githubusercontent.com/q3aql/ffmpeg-install/master/ffmpeg-install
```

2. 给脚本添加可执行权限

```
chmod a+x ffmpeg-install
```

3. 安装已发行版本的ffmpeg

```
./ffmpeg-install --install release
```

## 验证安装

最后，我们最好验证一下ffmpeg是否已经安装成功了。

```
ffmpeg -version
```