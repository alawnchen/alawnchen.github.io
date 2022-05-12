---
title: Atom和SSH客户端连接不上服务器的解决方法
date: 2018-10-28 08:15:59
tags: 
  - Atom
  - SSH代理
categories:
  - 技术
url: connecting-server-through-said-agent
---

> 最近，一台服务器连接不上了！服务器不在国内、不在亚洲，在遥远的澳洲。我经常用的Atom，偶尔会用来编辑点小文件，顺便上传到服务器，现在连接不上了！我经常用的Xshell，用来管理服务器，现在也连接不上了！巧了，有个朋友问了我怎么才能连接上连接不上的服务器，所以呢，在我解决了之后，顺便分享一下怎么解决！

<!--more-->

## 原因分析

服务器连接不上有各式各样的原因，不过大原因就一个，服务器和本地终端被阻隔了。所以呢，我们要做的就是在服务器和本地客户端之间搭个桥。原本服务器和本地客户端就像是分别在此岸和彼岸的两条路，我们架个桥，就可以把两岸的路连到一起了。知道了原因，我们接下来分步走！

## 代理

安装代理软件，我用的是shadowsocks，也是我推荐的，看自己习惯使用别的也可以（可以开启本地代理才行）。安装了，配置好，连接到服务器，并确保正常使用，这就完成了第一步。我们记住一下本地端口，默认的是1080。当然，这个可以修改。关于具体的使用，我就不多说了，大家可以搜索了解。

## windows实现

windows平台上面，socks5客户端很多选择，我在windows上面使用的是proxifier，也是我最推荐的一个。安装完成之后，我们启动软件之后，开始配置proxifier规则。我们依次打开proxifier的“配置文件”-“代理服务器”。在打开的代理服务器对话框，点击右边的“添加”按钮，打开另一个对话框。我们在地址一栏输入本地地址，即127.0.0.1，端口填写第一步的本地端口1080，在协议处选择SOCKS
Version 5。填好之后，依次保存即可。proxifier是个收费软件，大家可以找找免费的。

## Mac实现

Mac上面，我使用的是Proxychains-ng。这里简单说一下两种安装方式。最新的Release版本是4.13，在Github上面的地址是
https://github.com/rofl0r/proxychains-ng。

### 编译安装

```
wget https://github.com/rofl0r/proxychains-ng/archive/v4.13.tar.gz
tar xzf v4.13.tar.gz
cd proxychains-ng*
./configure --prefix=/usr --sysconfdir=/etc
make
make install
make install-config
```

### homebrew安装

```
brew install proxychains-ng
```

如果没有安装homebrew，自己安装一下。homebrew地址是https://brew.sh。推荐大家安装使用homebrew，是一个非常好的工具。

### 关闭SIP

macOS 10.11 后有一个功能叫做 SIP（System Integrity Protection）。Proxychains-ng代理需要我们把SIP关闭才能正常使用。关闭步骤如下：

 1. 重启Mac，按⌘ + R进入Recovery模式
 2. 依次选择顶部菜单的实用工具（Utilities）-> 终端（Terminal）
 3. 输入命令csrutil disable，并运行
 4. 重启系统后，终端里输入 csrutil status，如果成功关闭会显示System Integrity Protection
    status:disabled

### 配置Proxychains-ng

 * 编译安装时，配置文件路径是/etc/proxychains.conf
 * homebrew安装时，配置文件路径是/usr/local/etc/proxychains.conf

我们打开配置文件之后，找到[ProxyList]，在下面新添socks5  127.0.0.1 1080即可。

### 通过Proxychains-ng启动程序

```
proxychains 程序
```

顺便说一下启动终端启动atom。

```
ln -s /Applications/Atom.app/Contents/Resources/app/atom.sh /usr/local/bin/atom
proxychains atom
```

其他程序也是一样，平时怎么用程序就怎么用，在命令前面添加proxychains即可。

## Linux实现

Linux上面，推荐的是proxychains。Github地址是https://github.com/haad/proxychains。安装步骤如下：

```
wget https://github.com/haad/proxychains/archive/proxychains-4.2.0.tar.gz
cd proxychains-proxychains*
./configure
make
sudo make install
```

具体使用类似上面的proxychains-ng，好像proxychains-ng是基于proxychains开发的。这里就不多说了。

## 补充

proxifier有Mac版本，如果银子足够又不想折腾，这个就是最好的选择了！proxifier官网地址是https://www.proxifier.com