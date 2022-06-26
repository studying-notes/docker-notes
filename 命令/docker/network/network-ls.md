---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker network ls 命令"  # 文章标题
url:  "posts/docker/cmd/network/ls"  # 设置网页链接，默认使用文件名
tags: [ "docker", "docker-cli"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

> 显示一个或多个网络的详细信息

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network ls [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--no-trunc` | 不截断输出，即完整显示 `NETWORK ID` |
| `--quiet , -q` | 只显示 `NETWORK ID` |

## 扩展信息

- 列出 Engine `daemon` 知道的所有网络，包括跨集群中多个主机的网络。

## 示例

### 列出全部网络

```bash
$ sudo docker network ls
NETWORK ID          NAME                DRIVER          SCOPE
7fca4eb8c647        bridge              bridge          local
9f904ee27bf5        none                null            local
cf03ee007fb4        host                host            local
78b03ee04fc4        multi-host          overlay         swarm
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f type=custom -f type=builtin`，同时返回自定义和内建网络。

目前支持的过滤关键词有：

- driver
- id (network’s id)
- label (`label=<key>` or `label=<key>=<value>`)
- name (network’s name)
- scope (`swarm | global | local`)
- type (`custom | builtin | none`)

#### DRIVER

```bash
$ docker network ls --filter driver=bridge
NETWORK ID          NAME                DRIVER            SCOPE
db9db329f835        test1               bridge            local
f6e212da9dfd        test2               bridge            local
```

#### ID

```bash
$ docker network ls --filter id=63d1ff1f77b0
NETWORK ID          NAME                DRIVER           SCOPE
63d1ff1f77b0        dev                 bridge           local

$ docker network ls --filter id=95e
NETWORK ID          NAME                DRIVER          SCOPE
95e74588f40d        foo                 bridge          local
```

#### LABEL

```bash
$ docker network ls -f "label=usage"
NETWORK ID          NAME                DRIVER         SCOPE
db9db329f835        test1               bridge         local
f6e212da9dfd        test2               bridge         local
```

```bash
$ docker network ls -f "label=usage=prod"
NETWORK ID          NAME                DRIVER        SCOPE
f6e212da9dfd        test2               bridge        local
```

#### NAME

```bash
$ docker network ls --filter name=foobar
NETWORK ID          NAME                DRIVER       SCOPE
06e7eef0a170        foobar              bridge       local
```

```bash
$ docker network ls --filter name=foo
NETWORK ID          NAME                DRIVER       SCOPE
95e74588f40d        foo                 bridge       local
06e7eef0a170        foobar              bridge       local
```

#### SCOPE

```bash
$ docker network ls --filter scope=swarm
NETWORK ID          NAME                DRIVER              SCOPE
xbtm0v4f1lfh        ingress             overlay             swarm
ic6r88twuu92        swarmnet            overlay             swarm
```

```bash
$ docker network ls --filter scope=local
NETWORK ID          NAME                DRIVER              SCOPE
e85227439ac7        bridge              bridge              local
0ca0e19443ed        host                host                local
ca13cc149a36        localnet            bridge              local
f9e115d2de35        none                null                local
```

#### TYPE

```bash
$ docker network ls --filter type=custom
NETWORK ID          NAME                DRIVER       SCOPE
95e74588f40d        foo                 bridge       local
63d1ff1f77b0        dev                 bridge       local
```

**清理全部自定义网络**

```bash
$ docker network rm `docker network ls --filter type=custom -q`
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Network ID |
| `.Name` | Network name |
| `.Driver` | Network driver |
| `.Scope` | Network scope (local, global) |
| `.IPv6` | Whether IPv6 is enabled on the network or not. |
| `.Internal` | Whether the network is internal or not. |
| `.Labels` | All labels assigned to the network. |
| `.Label` | Value of a specific label for this network. For example `{{.Label "project.version"}}` |
| `.CreatedAt` | Time when the network was created |

```bash
$ docker network ls --format "{{.ID}}: {{.Driver}}"
afaaab448eb2: bridge
d1584f8dc718: host
391df270dc66: null
```
