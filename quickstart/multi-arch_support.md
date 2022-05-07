---
date: 2022-05-06T23:19:12+08:00
author: "Rustle Karl"

title: "跨系统架构构建与运行容器"
url:  "posts/docker/quickstart/multi-arch_support"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
series: [ "Docker 学习笔记" ]  # 系列
categories: [ "学习笔记" ]  # 分类

toc: true  # 目录
draft: false  # 草稿
---

> https://docs.docker.com/desktop/multi-arch/#build-multi-arch-images-with-buildx


## Windows

Windows 自带各种系统架构虚拟机。

## Linux

```shell
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
docker buildx rm builder
docker buildx create --name builder --driver docker-container --use
docker buildx inspect --bootstrap
```

## 列出支持的系统架构

```shell
docker buildx ls
```

```
NAME/NODE       DRIVER/ENDPOINT STATUS  PLATFORMS
desktop-linux   docker
  desktop-linux desktop-linux   running linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
default *       docker
  default       default         running linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

## 删除错误的构建

```shell
docker buildx rm build_name
```

## 运行指定系统架构的容器

### 默认与宿主机相同

```shell
docker run --rm debian uname -m
```

```
x86_64
```

### 指定哈希镜像

```shell
docker run --rm debian:latest@sha256:908d0c48bbcf433d9556407ba78bd971a938666eb7ec75292283f5ff10cf0a11 uname -m
```

```
armv7l
```
