---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network rm 命令"  # 文章标题
url:  "posts/docker/cmd/network/rm"  # 设置网页链接，默认使用文件名
tags: [ "docker", "docker-cli"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

> 移除一个或多个网络

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network rm NETWORK [NETWORK...]
```

## 扩展信息

- 在删除网络之前，必须断开连接到它的任何容器。

## 示例

### 删除网络

```bash
$ docker network rm my-network
```

### 删除多个网络

```bash
$ docker network rm 3695c422697f my-network
```

当指定了多个网络，Docker 会尝试一个一个删除，当一个失败，会继续下一个。
