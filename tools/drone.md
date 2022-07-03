---
date: 2022-06-27T20:54:25+08:00
author: "Rustle Karl"

title: "Drone 基于 Docker 的 CI/CD 工具"
url:  "posts/docker/tools/drone"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

## 与 Gitea 集成

> https://docs.drone.io/server/provider/gitea/

### 创建 OAuth2 应用程序

账户设置 -> 应用 -> 创建新的 OAuth2 应用程序

### Gitea 添加信任 WebHook

修改配置文件。

### 创建共享密钥

用于主节点服务器与工作节点间的通信认证。

```shell
openssl rand -hex 16
```

```
63365fa4bde1ef5355debf3b8de73872
```

### 下载

Drone 服务器是以 Docker 镜像方式发布的。

```shell
docker pull drone/drone
```

## 安装执行节点

https://docs.drone.io/runner/exec/installation/linux/

### 直接在本机上执行命令

#### 下载

```shell
curl -L https://github.com/drone-runners/drone-runner-exec/releases/latest/download/drone_runner_exec_linux_amd64.tar.gz | tar zx
```

#### 安装

```shell
install -t /usr/local/bin drone-runner-exec
```

arm64

```shell
curl -L https://github.com/drone-runners/drone-runner-exec/releases/latest/download/drone_runner_exec_linux_arm64.tar.gz | tar zx
install -t /usr/local/bin drone-runner-exec
```

```shell

```

#### 配置

```shell
mkdir /etc/drone-runner-exec
```

```shell
vim /etc/drone-runner-exec/config
```

```shell
DRONE_RPC_SECRET=63365fa4bde1ef5355debf3b8de73872
DRONE_RPC_PROTO=http
DRONE_RPC_HOST=192.168.0.16:11280
DRONE_LOG_FILE=/var/log/drone-runner-exec/runner.log
```

```shell
mkdir /var/log/drone-runner-exec
```

#### 启动

```shell
drone-runner-exec service install
```

```shell
drone-runner-exec service start
```

```shell
tail -f /var/log/drone-runner-exec/runner.log
```

## 全过程参考

https://zhuanlan.zhihu.com/p/467226649

## 非管理员的问题

除非首次，否则直接修改数据库，不然不生效。

```shell

```
