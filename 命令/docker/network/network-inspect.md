---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network inspect 命令"  # 文章标题
url:  "posts/docker/cmd/network/inspect"  # 设置网页链接，默认使用文件名
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

> 显示一个或多个网络的详细信息

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--format, -f` | 用给定的 Go 模板格式化输出 |
| `--verbose , -v` | 详细输出以进行诊断 |

## 扩展信息

- 默认情况下，该命令将所有结果呈现在 JSON 对象中。
