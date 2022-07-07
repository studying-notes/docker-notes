---
date: 2022-06-26T13:55:07+08:00
author: "Rustle Karl"

title: "Harbor 搭建企业级 Docker Registry 服务"
url:  "posts/docker/tools/harbor"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

[Harbor](https://goharbor.io/) 是 `VMware` 公司开源了企业级 `Registry` 项目, 其的目标是帮助用户迅速搭建一个企业级的 `Docker registry` 服务。

由于 Harbor 是基于 Docker Registry V2 版本，所以 docker 版本必须 `>=1.10.0` [docker-compose](https://docs.docker.com/compose/install/#prerequisites) `>=1.6.0`

Github: [goharbor/harbor](https://github.com/goharbor/harbor)，官方[预览示例](https://demo.goharbor.io/)

对硬件需求

> CPU  => 最小 2CPU/4CPU(首选)  
> Mem  => 最小 4GB/8GB(首选)  
> Disk => 最小 40GB/160G(首选)  

## 下载安装包

可以从[发布页面](https://github.com/goharbor/harbor/releases)下载安装程序的二进制文件，选择在线或离线安装程序，使用 tar 命令解压缩包。

```bash
# 下载离线安装包
wget https://github.com/goharbor/harbor/releases/download/v2.5.2/harbor-offline-installer-v2.5.2.tgz -O harbor.tgz
```

```bash
# 解压缩包
tar xvf harbor.tgz
```

## 修改配置

```bash
cp harbor.yml.tmpl harbor.yml
```

```bash
vim harbor.yml
```

修改文件相关配置。

## 运行安装脚本

```bash
./install.sh
```

要更改 Harbour 的配置，请先停止现有的 Harbor 实例并更新 `harbour.yml`。 然后运行 `prepare` 脚本来填充配置，最后重新创建并启动 Harbor 的实例：

```bash
docker-compose down -v
```

```bash
vim harbor.yml
```

```bash
./prepare
```

```bash
docker-compose up -d
```

## 使用 harbor 

```bash
docker login 192.168.188.222:8070
```

```bash
docker pull nginx
docker tag nginx 192.168.188.222:8070/library/nginx:latest
docker push 192.168.188.222:8070/library/nginx
docker rmi -f 192.168.188.222:8070/library/nginx:latest
```

若推送镜像报以下错误:

> Error response from daemon: Get https://192.168.188.222:8070/v1/users/: http: server gave HTTP response to HTTPS client

原因为，`docker` 默认使用的是 `https` 协议，而搭建的 `Harbor` 是 `http` 提供服务的，所以要配置可信任。

客户机 docker 启动时候带上 `--insecure-registry=docker.xxx.com` 强制 `docker login` 走 `http` 的 `80` 端口，就可以正常 `push` 了。

## 功能简介

Harbor 大概有项目、日志、用户管理、仓库管理、同步管理、配置管理和垃圾清理等功能。

### 项目

就是之前说的镜像仓库，可以创建公开和私有的镜像仓库，公开的项目无需验证即可拉取镜像，但是要推送镜像的话还是需要验证的；私有镜像任何操作都需要验证。

### 日志

记录用户对项目的所有创建、删除、修改等操作。

### 仓库管理

Harbor 支持多个 Harbor 仓库之间的镜像同步，在仓库管理中可以添加其他的 Harbor 仓库。

### 同步管理

用来编写多个 Harbor 镜像仓库之间的同步规则。以便更好的管理多个 Harbor 仓库

### 配置管理

可以修改 Harbor 仓库的便签、认证模式、是否允许用户自注册、邮箱服务器配置、创建项目的权限和仓库属性和令牌过期时间等。

默认的公开仓库 library 中，需要推送镜像是需要认证之后才可以的，拉取镜像则无需认证。

关于认证就是需要在 Harbor 用户管理中添加用户并设置密码，然后在 docker 客户端中通过 `docker login` 登陆，只有登陆成功之后才算完成了验证。并且不同的 Harbor 项目只有拥有权限的成员才可以做相关操作。

## 缓存代理

https://goharbor.io/docs/2.5.0/administration/configure-proxy-cache/


```bash
docker pull <harbor_server_name>/<proxy_project_name>/goharbor/harbor-core:dev
```

无法透明代理，必须加上前缀，还是不太方便。

不过搭配 K3s 的重写，可以实现透明代理。
