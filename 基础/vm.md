---
date: 2020-12-29T14:52:48+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker 与虚拟机"  # 文章标题
# description: "文章描述"
url:  "posts/abc/vm"  # 设置网页永久链接
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

Docker 不是虛拟机，容器中的应用都应该以前台执行，而不是像虛拟机、物理机里面那样启动后台服务，容器内没有后台服务的概念。对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退容器就失去了存在的意义，从而退出。其它辅助进程不是它需要关心的东西。
