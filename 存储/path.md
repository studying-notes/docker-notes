---
date: 2021-01-04T17:51:28+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "修改 Docker 镜像存储位置"  # 文章标题
url:  "posts/docker/storage/path"  # 设置网页链接，默认使用文件名
tags: [ "docker", "storage"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

```shell
systemctl stop docker
```

## 通过修改配置文件

```shell
mkdir -p /mnt/storage/system/docker
```

```shell
cp -R /var/lib/docker/* /mnt/storage/system/docker
```

```ini
/etc/docker/daemon.json
```

```json
{
  "data-root": "/mnt/storage/system/docker",
}
```

```shell
systemctl restart docker
```

```shell
docker info
```

```
Docker Root Dir: /mnt/storage/system/docker
```

```shell

```

```shell

```
