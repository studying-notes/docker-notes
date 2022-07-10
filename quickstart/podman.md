---
date: 2022-07-10T23:07:04+08:00
author: "Rustle Karl"

title: "Podman 配置与使用"
url:  "posts/docker/quickstart/podman"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

## 安装

参考官方文档：

https://podman.io/getting-started/installation.html

```shell
sudo apt-get -y install podman
```

```shell
opkg install crun podman
```

## 配置

### 仓库

```shell
vim /etc/containers/registries.conf
```

```conf
unqualified-search-registries = ['docker.io', 'gcr.io']

[[registry]]
prefix = "docker.io"
location = "192.168.0.18:12480/proxy.docker.io"
insecure = true

[[registry]]
prefix = "gcr.io"
location = "192.168.0.18:12480/proxy.gcr.io"
insecure = true
```

修改后不需要重启，可以自定义前缀，配合 Harbor 非常不错。

这两点比 Docker 好。

测试：

```shell
podman pull postgres:alpine
```

与 Docker 应该不冲突，可以一起试着用。

## 二级

### 三级

```shell

```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


## 二级

### 三级

```shell

```

```shell

```


