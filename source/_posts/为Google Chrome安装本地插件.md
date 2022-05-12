---
title: 为Google Chrome安装本地插件
date: 2018-10-28 22:20:13
tags: 
  - google chrome
categories:
  - 记录
url: install-google-chrome-local-extensions
---

> 很早就开始使用Google Chrome浏览器，除了我自己对Google的产品有一种迷之好感，还有Google Chrome秉承的简洁高效理念以及Google Chrome自带的安全功能。

<!--more-->

## 安全和自定义的矛盾

产品的安全，最好是稳定地开启安全设置项，也就是说最好让官方尽可能地封堵上可能惹上安全问题的功能。而Google就是以用户安全为前提，最大限度地去限制了Google
Chrome使用本地扩展。本地安装插件确实是方便了自定义，但却可能会引发未知的安全问题，毕竟信手拈来的代码没有经过官网的审核很可能会有问题。所以，安全和自定义是一个矛盾！

## Windows下安装本地扩展

我们要把拿到的代码打包成扩展，并且确保本地安装的Google Chrome扩展安全稳定地运行！

 1. 打开Google
    Chrome插件管理页面，即chrome://extensions，在右上角处打开“开发者模式”。如果打开成功，那么左上角会出现“加载已解压的扩展程序”和“打包扩展程序”两个按钮
 2. 点击“打包扩展程序”按钮，选择要打包的文件夹，按照提示把代码打包成扩展程序。把打包好的本地扩展程序crx拖拽到扩展程序页面完成安装。安装后，扩展程序无法启用。找到安装好的扩展程序，我们复制一下扩展程序的ID。
 3. 打开http://www.chromium.org/administrators/policy-templates  页面下载策略模版（[点击这里](https://dl.google.com/dl/edgedl/chrome/policy/policy_templates.zip)下载也可以）。解压之后，在windows文件夹里面打开adm文件夹，找到自己系统语言对应的文件夹，把chrome.adm解压出来。点击开始菜单，打开运行，在输入框里面输入gpedit.msc，按enter启动本地组策略编辑器。
 4. 我们在左边展开“计算机配置”，在“管理模版”处右击，选择“添加/删除模版(A)…”打开“添加/删除模版”对话框。在对话框里面，点击“添加”按钮，选择上一步得到的chrome.adm文件，导入管理模版。
 5. 导入成功后，依次展开“计算机配置”-“管理模版”-“经典管理模版ADM”-“Google”-“Google Chrome”-“扩展程序”，在右侧，我们双击“配置扩展程序安装白名单”，打开“配置扩展程序安装白名单”对话框。点击“启用”后，下面会显示内容，在下面左边点击“显示”按钮，把第2步得到的ID添加进去，然后依次保存。
 6. 重启Google Chrome，会有一个提示，但是也是最后的提示。此后就可以正常地使用这个本地扩展程序，就像在线安装的一样。

## Mac下安装本地扩展

打包步骤，参考上面的第2步和第2步即可，而且不需要任何的配置了！

## Linux下安装本地扩展

打包步骤也是参考windows下的第1和第2步骤，具体是不是需要配置，我就没有去实践了，可打开<http://www.chromium.org/administrators/policy-templates>看看说明！