---
date: 2020-12-29T14:28:10+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker 容器自动重启"  # 文章标题
url:  "posts/docker/turtial/restart"  # 设置网页链接，默认使用文件名
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

## restart 参数

- no : 容器退出时，不重启容器；
- on-failure : 只有在非 0 状态退出时才从新启动容器；
- always : 无论退出状态是如何，都重启容器；

在使用 on-failure 策略时，指定 Docker 将尝试重新启动容器的最大次数。默认情况下，Docker 将尝试永远重新启动容器。

```shell
docker run --restart=on-failure:5 <image>
```

## 创建容器时指定参数

在 `docker run` 指令中加入 `--restart=always`。

```shell
docker run --restart=always -d -p 8080:8080 swaggerapi/swagger-editor
```

## 创建容器后更新参数

```shell
docker update --restart=always <container_id>
```
