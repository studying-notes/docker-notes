---
date: 2022-06-27T12:43:36+08:00
author: "Rustle Karl"

title: "AutoK3s 介绍"
url:  "posts/docker/docs/k3s/autok3s"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

> https://docs.rancher.cn/docs/k3s/autok3s/_index

AutoK3s 是用于简化 K3s 集群管理的轻量级工具，您可以使用 AutoK3s 在任何地方运行 K3s 服务。

## 关键特性

- 通过 API、CLI 和 UI 等方式快速创建 K3s。
- 云提供商集成（简化 CCM 设置）。
- 灵活安装选项，例如 K3s 集群 HA 和数据存储（内置 etcd、RDS、SQLite 等）。
- 低成本（尝试云中的竞价实例）。
- 通过 UI 简化操作。
- 多云之间弹性迁移，借助诸如 backup-restore-operator 这样的工具进行弹性迁移。

## 快速体验

### Docker

您可以通过以下 Docker 命令，一键启动 AutoK3s 本地 UI，快速体验相关功能。

```shell
docker run --rm -p 8080:8080 cnrancher/autok3s
```

如果您想要在 docker 中使用 K3d provider，那么您需要使用宿主机网络启动 AutoK3s 镜像。

```shell
docker run --rm --net host -v /var/run/docker.sock:/var/run/docker.sock cnrancher/autok3s
```

### 原生安装

如果您是 MacOS 或者 Linux 系统，您也可以使用以下安装命令，一键安装 AutoK3s（Windows用户请前往 Releases 页面下载对应的程序）。

```shell
curl -sS https://rancher-mirror.rancher.cn/autok3s/install.sh  | INSTALL_AUTOK3S_MIRROR=cn sh
```

您可以通过以下 CLI 命令启动本地 UI。

```shell
autok3s serve
```

```shell
autok3s serve --bind-address 0.0.0.0 --bind-port 18480
```

您也可以通过以下 CLI 在 AWS EC2 快速创建一个 1 master, 1 worker 节点的 K3s 集群。

```shell
export AWS_ACCESS_KEY_ID='<Your access key ID>'
export AWS_SECRET_ACCESS_KEY='<Your secret access key>'

autok3s -d create -p aws --name myk3s --master 1 --worker 1
```

## 卸载

```shell
/usr/local/bin/autok3s-uninstall.sh
```

## 升级

如果您使用 Docker 命令一键启动 AutoK3s 本地 UI，从 v0.4.0 升级到 v0.5.1 需要进行如下操作以保证历史数据的迁移。

```shell
docker cp <old-container>:/root/.autok3s .
docker rm -f <old-container>
docker run -itd --restart=unless-stopped -p 8080:8080 -v $PWD/.autok3s:/root/.autok3s cnrancher/autok3s:v0.5.1
```

在 v0.4.1 以后的版本，可以直接通过 --volumes-from 来保证历史数据的迁移。

```shell
docker stop <old-container>
docker run -itd --restart=unless-stopped -p 8080:8080 --volumes-from <old-container> cnrancher/autok3s:v0.5.1
```

## 注意点

不能即是主节点，又是从节点。否则报错：

```
master node internal ip address can not be empty
```

## 查询节点

```shell
autok3s kubectl get nodes
```

```shell

```
