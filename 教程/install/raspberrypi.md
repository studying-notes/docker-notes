---
date: 2021-01-01T17:36:09+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "树莓派安装 Docker"  # 文章标题
url:  "posts/docker/turtial/install/raspberrypi"  # 设置网页链接，默认使用文件名
tags: [ "raspberrypi", "docker"]  # 标签
series: [ "树莓派学习笔记",  "Docker 从入门到放弃" ]  # 系列
categories: [ "学习笔记"]  # 分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 一键安装

```shell
curl -sSL https://get.docker.com | sh
```

> 在修改镜像源之前

```shell
docker run hellow-world
```

> 不限系统，可以是树莓派官方，也可以是 Ubuntu。

```shell
usermod -aG docker pi
```

## 检查版本

```shell
docker version
```

## 升级

```shell
apt-get upgrade
```

## 卸载

```shell
apt-get purge docker-ce
```

```shell
apt-get purge docker-ce-cli
```

```shell
rm -rf /var/lib/docker
```

## 修改镜像源和配置等

### 修改配置文件

```shell
echo '
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
  "experimental": true
}
' > /etc/docker/daemon.json
```

### 使配置文件生效

```shell
systemctl daemon-reload
```

### 重启 Docker

```shell
systemctl restart docker
```

```shell

```


## 安装 Docker Compose

```shell
pip install docker-compose
```

```shell

```

```shell

```
