---
title: Linux下echo+tee实现同时追加或者覆盖多个文本文件
date: 2018-10-28 09:44:46
tags: 
  - echo
  - tee
categories:
  - 技术
url: operate-many-files-in-echo-and-tee-on-linux
---

> Linux是个美妙的操作系统，它高效、它安全、它简洁，它还有着神秘的魅力！不同于Windows的图形化的直白，Linux和macOS的美妙之处总是需要大伙去用心去发现。

<!--more-->


向文件里面追加或者覆盖一些文本内容，我们最容易想到也是最容易选择到的命令是echo命令。

 ## echo追加内容

```
echo 'test' >> test.txt
```


 ## echo覆盖内容

```
echo 'test' > test.txt
```

确实很简单好用，也没有什么繁复的参数。前提是一个文件，如果要求同时向多个文件追加相同的内容呢？echo命令就会显得力不从心！！而利用echo+tee命令就可以完成同时追加或者覆盖相同内容到多个文件的需求！

### echo+tee同时追加

```
echo 'test' | tee -a file1 file2 file3
```


## echo+tee同时覆盖

```
echo 'test' | tee file1 file2 file3
```


echo+tee命令的妙用就在于，可以输出的同时，一次性同时追加或者覆盖相同的文本内容到不同的文件。如果你也有这样的需求，不妨试试看看！！