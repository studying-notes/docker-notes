---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network 网络命令"  # 文章标题
url:  "posts/docker/cmd/network"  # 设置网页链接，默认使用文件名
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

> 管理网络

## 基本信息

### 用法

```
docker network COMMAND
```

### 命令

{{ <link src="posts/docker/命令/docker/network/network-connect"> }}
{{ <link src="posts/docker/命令/docker/network/network-create"> }}
{{ <link src="posts/docker/命令/docker/network/network-disconnect"> }}
{{ <link src="posts/docker/命令/docker/network/network-inspect"> }}
{{ <link src="posts/docker/命令/docker/network/network-ls"> }}
{{ <link src="posts/docker/命令/docker/network/network-prune"> }}
{{ <link src="posts/docker/命令/docker/network/network-rm"> }}

| 命令 | 描述 |
| ------------- | ------------- |
| [docker network connect](https://docs.docker.com/engine/reference/commandline/network_connect/) | 将容器连接到网络 |
| [docker network create](https://docs.docker.com/engine/reference/commandline/network_create/) | 创建网络 |
| [docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/) | 断开容器与网络的连接 |
| [docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/) | 显示一个或多个网络的详细信息 |
| [docker network ls](https://docs.docker.com/engine/reference/commandline/network_ls/) | 列出网络 |
| [docker network prune](https://docs.docker.com/engine/reference/commandline/network_prune/) | 移除全部未使用网络 |
| [docker network rm](https://docs.docker.com/engine/reference/commandline/network_rm/) | 移除一个或多个网络 |
