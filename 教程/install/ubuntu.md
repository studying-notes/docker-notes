---
date: 2020-12-29T14:19:21+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Ubuntu 下安装 Docker 全过程"  # 文章标题
url:  "posts/docker/turtial/install/ubuntu"  # 设置网页链接，默认使用文件名
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

## 卸载原来的版本

```shell
apt-get remove docker docker-engine docker-ce docker.io
```

## 安装依赖

```shell
apt-get update
```

```shell
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

## 添加源证书

```shell
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

```shell
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

```shell
apt-get update
```

## 安装 Docker

```shell
apt-get install -y docker-ce docker-ce-cli containerd.io
```

## 进入 root 组

```shell
sudo usermod -aG docker <user>
```

```shell
sudo usermod -aG docker node1
```

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

## 开机启动

```shell
systemctl enable docker.service
```

```shell
systemctl restart docker.service
```

### 测试

```shell
docker run --rm hello-world
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

```shell

```

```shell

```
