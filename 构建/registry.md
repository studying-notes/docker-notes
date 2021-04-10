---
date: 2020-12-29T15:29:23+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "自建私有 Docker 仓库 Registry"  # 文章标题
# description: "文章描述"
url:  "posts/docker/build/registry"  # 设置网页永久链接
tags: [ "docker", "docker-build"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 拉取镜像

```shell
docker run -d -p 5000:5000 --restart=always registry
```

```shell
docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
```

```shell
docker push 127.0.0.1:5000/ubuntu:latest
```

```shell
curl  127.0.0.1:5000/v2/_catalog
```

```shell
docker image rm 127.0.0.1:5000/ubuntu:latest
```

```shell
docker pull 127.0.0.1:5000/ubuntu:latest
```

## 通过修改 hosts

`/etc/hosts`

```ini
192.168.1.5 local
```

刷新 DNS

```shell
/etc/init.d/nscd restart
```

`/etc/docker/daemon.json`

```bash
echo '
{
    "exec-opts": [
        "native.cgroupdriver=systemd"
    ],
    "registry-mirrors": [
        "http://node1:5000",
        "http://ovfftd6p.mirror.aliyuncs.com",
        "http://registry.docker-cn.com",
        "http://docker.mirrors.ustc.edu.cn",
        "http://hub-mirror.c.163.com"
    ],
    "insecure-registries": [
        "registry.docker-cn.com",
        "node1:5000",
        "docker.mirrors.ustc.edu.cn"
    ]
}
' > /etc/docker/daemon.json && systemctl restart docker
```
```shell
systemctl restart docker
```

```shell
docker tag ubuntu:latest local:5000/ubuntu:latest
```

```shell
docker pull node1:5000/comic-scrapy-render
docker push local:5000/ubuntu:latest
```

```shell
curl local:5000/v2/_catalog
```


```shell

```

```shell

```
