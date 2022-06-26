---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network prune 命令"  # 文章标题
url:  "posts/docker/cmd/network/prune"  # 设置网页链接，默认使用文件名
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

> 移除全部未使用网络

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network prune [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter` | 指定过滤条件 |
| `--force , -f` | 不提示是否确认，强制删除 |

## 扩展信息

- 未使用的网络是指未被任何容器引用的网络。

## 示例

### 删除网络

```bash
$ docker network prune

WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Networks:
n1
n2
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

- until (`<timestamp>`) - only remove containers created before given timestamp
- label (`label=<key>`, `label=<key>=<value>`, `label!=<key>`, or `label!=<key>=<value>`) - only remove containers with (or without, in case `label!=...` is used) the specified labels.

```bash
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
7430df902d7a        bridge              bridge              local
ea92373fd499        foo-1-day-ago       bridge              local
ab53663ed3c7        foo-1-min-ago       bridge              local
97b91972bc3b        host                host                local
f949d337b1f5        none                null                local

$ docker network prune --force --filter until=5m

Deleted Networks:
foo-1-day-ago

$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
7430df902d7a        bridge              bridge              local
ab53663ed3c7        foo-1-min-ago       bridge              local
97b91972bc3b        host                host                local
f949d337b1f5        none                null                local
```
