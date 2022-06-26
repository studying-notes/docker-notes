---
date: 2020-12-29T14:19:21+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Windows 下安装 Docker 全过程"  # 文章标题
url:  "posts/docker/turtial/install/windows"  # 设置网页链接，默认使用文件名
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

## 下载地址

```shell
https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe
```

```shell
https://desktop.docker.com/win/stable/Docker%20Desktop%20Installer.exe
```

## 配置内容

```json
{
  "registry-mirrors": [
    "http://ovfftd6p.mirror.aliyuncs.com",
    "http://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ],
  "insecure-registries": [
    "registry.docker-cn.com",
    "docker.mirrors.ustc.edu.cn"
  ],
  "debug": true,
  "experimental": false
}
```

```shell
{
    "exec-opts": [
        "native.cgroupdriver=systemd"
    ],
    "registry-mirrors": [
        "http://ovfftd6p.mirror.aliyuncs.com",
        "http://registry.docker-cn.com",
        "http://docker.mirrors.ustc.edu.cn",
        "http://hub-mirror.c.163.com"
    ],
    "insecure-registries": [
        "registry.docker-cn.com",
        "docker.mirrors.ustc.edu.cn"
    ]
}
```

### 添加私有 registry

```json
{
  "registry-mirrors": [
    "http://node1:5000",
  ],
  "insecure-registries": [
    "node1:5000",
  ],
  "debug": true,
  "experimental": false
}
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
