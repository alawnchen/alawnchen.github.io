---
title: iOS设备webview弹窗里iframe的可滚动方式
date: 2017-11-17 08:20:00
tags: 
  - 前端
  - iframe
  - 弹窗
categories:
  - 技术
url: rolling-iframe-webpage-content-in-webview-of-ios
---

> 最近公司休假，所以为了方便，公司的网站需要搞一个公告，用以提醒网站会员。而为了美观，我选择了弹窗里面嵌入iframe的方式作为展示公告的方式。我原以为很简单的，却没想到，我花了几个小时才处理好这个原本预期10分钟可以解决的问题。所以，把这个坑记一下！
样式化iframe父级容器
弹窗里面的iframe必须有一个div外层包住，并给其样式如下：

```
width:100%;height:258px;overflow:auto!important; -webkit-overflow-scrolling:touch!important;
```


当然，具体的样式还得自己调！不过呢，这里的每一项都必须明确，且除了高宽，其他的都必须是现在对应的值！

## 样式化iframe

iframe的样式化是必须的了，iframe样式化的样式如下：

```
width:100%;height:258px;display:block;overflow:scroll;-webkit-overflow-scrolling:touch;
```


和上面的父级容器一样，这里的每一项都必须明确，且除了高宽，其他的都必须是现在对应的值！所不同的是权限，这个得看清楚。

## iframe内容样式化

iframe里面内容并不需要什么特殊的样式化，只需要设置高和宽都是100%即可！

## 结语

有时候，iOS也是可以坑死人的，是不是？所以，记一下，还是必要的！

