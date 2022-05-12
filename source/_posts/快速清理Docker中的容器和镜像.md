---
title: 快速清理Docker中的容器和镜像
date: 2018-10-29 06:48:47
tags: 
  - Docker
categories:
  - 技术
url: quickly-delete-dockers-containtor-and-images
---

> Docker用起来很爽，但是如果不熟悉，也会很棘手！比如：怎么让Docker中的镜像随机启动，又或者怎么快速地删除镜像。下面是Docker中清理容器和镜像的命令！

<!--more-->


## 停止所有容器的进程

```
docker kill $(docker ps -a -q)
```


## 删除所有已经停止的容器

```
docker rm $(docker ps -a -q)
```


## 删除所有未打 dangling 标签的镜像

```
docker rmi $(docker images -q -f dangling=true)
```


## 删除所有镜像

```
docker rmi $(docker images -q)
```


## 为这些命令创建别名

```
vi ~/.bash_aliases
停止所有容器的进程.
alias dockerkill="docker kill $(docker ps -a -q)"
删除所有已经停止的容器.
alias dockercleanc="docker rm $(docker ps -a -q)"
删除所有未打标签的镜像.
alias dockercleani="docker rmi $(docker images -q -f dangling=true)"
删除所有已经停止的容器和未打标签的镜像.
alias dockerclean="dockercleanc || true && dockercleani"
```

