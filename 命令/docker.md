---
date: 2020-12-29T14:54:42+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker 常用命令集合"  # 文章标题
# description: "文章描述"
url:  "posts/cmd/docker"  # 设置网页永久链接
tags: [ "docker"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 保存镜像

```shell
docker save -o photoprism-opencv.tar photoprism:opencv
```

## 载入镜像

```shell
docker load < photoprism-opencv.tar
```

## 列出所有的容器 ID

```shell
docker ps -aq  # 包括已停止的容器
```

```shell
docker ps -q  # 运行状态的容器
```

```shell
docker ps | awk '{print $2,$NF}'  # 两列
```

## 停止所有的容器

```shell
docker stop $(docker ps -q)
```

## 删除所有的容器

```shell
docker rm $(docker ps -aq)
```

## 删除所有的镜像

```shell
docker rmi $(docker images -q)
```

## 提取容器运行状态

```shell
docker inspect -f {{.State.Status}} nginx
```

## 提取容器 IP

```shell
docker inspect -f {{.NetworkSettings.IPAddress}} nginx
```

## 提取日志

```shell
docker logs nginx
```

## 实时日志

```shell
docker logs nginx -f
```

## 搜索镜像

```shell
docker search busybox
```

## 拉取镜像

```shell
docker pull busybox:latest
```

```shell
docker pull alpine:latest
```

## 镜像信息

```shell
docker image ls busybox
```

## 启动镜像终端

```shell
docker run -it busybox
```

## 创建数据卷

```shell
docker volume create [xxx]
```

## 查看所有的数据卷

```shell
docker volume ls
```

## 查看指定数据卷的信息

```shell
docker volume inspect [xxx]
```

## 删除数据卷

```shell
docker volume rm [xxx]
```

## 清除无主数据卷

```shell
docker volume prune
```

## 终止状态的容器

```shell
docker container ls -a
```

## 重新启动终止状态的容器

```shell
docker restart container
```

```shell
docker start container
```

## 查看摘要

```shell
docker run -dit ubuntu
```

## 在容器运行时进入终端，exit 导致容器停止

```shell
docker attach [container]
```

## 在容器运行时进入终端，exit 容器不停止

```shell
docker exec -it [container] bash
```

## 导出容器快照到本地文件

```shell
docker export [container] > xxx.tar
```

## 导入容器快照

> 快照文件丢弃所有历史记录和元数据信息

```shell
cat xxx.tar | docker import -test/xxx:v1.0
```

```shell
docker import xxx.tar
```

## 删除容器

```shell

```

## 显示悬挂镜像

```shell
docker images --filter "dangling=true"
```

## 删除所有悬挂镜像

```shell
docker images --filter "dangling=true" -q | xargs docker rmi
```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```

```shell

```
