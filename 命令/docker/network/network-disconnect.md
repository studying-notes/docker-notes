---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network disconnect 命令"  # 文章标题
url:  "posts/docker/cmd/network/disconnect"  # 设置网页链接，默认使用文件名
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

> 断开容器与网络的连接

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network disconnect [OPTIONS] NETWORK CONTAINER
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--force , -f` | 强制容器断开网络连接 |

## 扩展信息

- 容器必须正在运行才能将其与网络断开连接。

## 示例

```bash
$ docker network disconnect multi-host-network container1
```
