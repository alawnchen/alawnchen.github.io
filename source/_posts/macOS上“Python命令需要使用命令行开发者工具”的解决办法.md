---
title: macOS上“Python命令需要使用命令行开发者工具”的解决办法
date: 2022-07-15 02:37:38
tags: 
  - macOS
  - node-sass
  - vue
  - xcode
categories:
  - 技术
photos:
  - "/images/624d8ab48e0078684166f018_ios_code_signing_bitrise.png"
url: python-commands-require-the-command-line-developer-tools
description: 最近开发一个项目使用到了买回来的一个vue模板，而模板使用到了scss这个css的解析器。而我在安装依赖时候总是遇到很奇怪的问题。
---

> 最近开发一个项目使用到了买回来的一个vue模板，而模板使用到了scss这个css的解析器。而我在安装依赖时候总是遇到很奇怪的问题。

## 安装开发者工具后无效

npm install命令开始安装依赖之后，总是会卡在node-sass依赖导致的一系列gyp ERR错误。而程序也会自动调用xcode-select命令唤起依赖的引导安装程序。确认安装之后，经过十几分钟的等待，确确实实地收到了安装完成的提示后，重新执行npm install命令后，依旧会遇到这个问题。遇到这个情况，我一开始猜想可能是什么环境变量需要重启一下，所以，我重启了电脑之后，打开webstorm，在内置终端敲上npm install之后还是会收到和之前一样的情况。

## 手动安装python无效

在上面安装开发者工具后无效后，我尝试使用brew安装python2到系统上。我在/etc/paths最后一行加上了python的安装目录，这一步是为了将python的安装目录设置成环境变量，以便可以直接使用python命令。安装好python之后，我走上一步的npm install安装依赖，依旧还是提示需要安装命令行开发者工具。至此，手动安装这条路也被证实不可用。

## 建立python3软连接

在手动安装python被证实无效后，我想到系统里面已经有python3，那我直接在python3基础上建立软连接也许可行。于是，我用which命令找到python3的路径后，直接在同目录下以python3路径为源，建立了一个指向python的软连接。建立好连接后，我又走上一步的npm install安装依赖，但我期待的结果依旧没有出现，还是提示需要安装命令行开发者工具。至此，建立python3软连接也被证实不可用。

## 解决办法

弄了几步，不得其要，让人挺烦的。Google了一下，找到一堆乱七八糟的答案。一一试了，每次开始都是满怀希望，结果都是大失所望。最后，在stackoverflow找到一条答案说，是在开发者工具包/usr/bin文件夹里面创建一个python的软连接，指向同目录的python3即可。死马当活马医了，我输入sudo ln -s $(xcode-select --print-path)/usr/bin/python3 $(xcode-select --print-path)/usr/bin/python后回车。抱着试一试不行再想办法的心，继续一开始的npm install，终于这回不再提示需要安装开发者工具。