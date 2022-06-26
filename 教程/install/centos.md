---
date: 2021-02-22T16:15:38+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "CentOS 下安装 Docker 全过程"  # 文章标题
url:  "posts/docker/turtial/install/centos"  # 设置网页链接，默认使用文件名
tags: [ "docker"]  # 自定义标签
series: [ "Docker 从入门到放弃" ]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 手动安装

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

```shell
yum install -y yum-utils
```

```shell
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```shell
yum install docker-ce docker-ce-cli containerd.io
```

```shell
usermod -aG docker root
```

```shell
systemctl enable docker.service
```

```shell
systemctl restart docker.service
```

## 安装 Compose

Python 安装见 Python 笔记

```shell
curl -L "https://github.com/docker/compose/releases/download/1.28.4/docker-compose-linux-x86_64"
```

```shell
cp docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
```

```shell
chmod +x /usr/local/bin/docker-compose
```

```shell
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

```shell
usermod -aG docker root
```

curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

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
  ]
}
' > /etc/docker/daemon.json
```
