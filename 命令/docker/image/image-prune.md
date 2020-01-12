# `docker image prune` 命令详解

> 移除未使用镜像

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker image prune [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--all , -a` | 移除全部未使用镜像，默认只移除悬空镜像 |
| `--filter` | 指定过滤条件 |
| `--force , -f` | 不提示是否确认，强制删除 |

## 扩展信息

- 反向过滤时，由于 `docker image ls` 不支持反向过滤，所以无法预测哪些镜像会被删除

## 示例

### 删除镜像

```bash
$ docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

- until (`<timestamp>`) - only remove containers created before given timestamp
- label (`label=<key>`, `label=<key>=<value>`, `label!=<key>`, or `label!=<key>=<value>`) - only remove containers with (or without, in case `label!=...` is used) the specified labels.

```bash
docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.CreatedAt}}\t{{.Size}}'
REPOSITORY          TAG                 IMAGE ID            CREATED AT                      SIZE
foo                 latest              2f287ac753da        2017-01-04 13:42:23 -0800 PST   3.98 MB
alpine              latest              88e169ea8f46        2016-12-27 10:17:25 -0800 PST   3.98 MB
busybox             latest              e02e811dd08f        2016-10-07 14:03:58 -0700 PDT   1.09 MB

$ docker image prune -a --force --filter "until=2017-01-04T00:00:00"

Deleted Images:
untagged: alpine:latest
untagged: alpine@sha256:dfbd4a3a8ebca874ebd2474f044a0b33600d4523d03b0df76e5c5986cb02d7e8
untagged: busybox:latest
untagged: busybox@sha256:29f5d56d12684887bdfa50dcd29fc31eea4aaf4ad3bec43daf19026a7ce69912
deleted: sha256:e02e811dd08fd49e7f6032625495118e63f597eb150403d02e3238af1df240ba
deleted: sha256:e88b3f82283bc59d5e0df427c824e9f95557e661fcb0ea15fb0fb6f97760f9d9

Total reclaimed space: 1.093 MB

$ docker images --format 'table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.CreatedAt}}\t{{.Size}}'

REPOSITORY          TAG                 IMAGE ID            CREATED AT                      SIZE
foo                 latest              2f287ac753da        2017-01-04 13:42:23 -0800 PST   3.98 MB
```

```bash
$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
foo                 latest              2f287ac753da        14 seconds ago      3.98 MB
alpine              latest              88e169ea8f46        8 days ago          3.98 MB
debian              jessie              7b0a06c805e8        2 months ago        123 MB
busybox             latest              e02e811dd08f        2 months ago        1.09 MB
golang              1.7.0               138c2e655421        4 months ago        670 MB

$ docker image prune -a --force --filter "until=240h"

Deleted Images:
untagged: golang:1.7.0
untagged: golang@sha256:6765038c2b8f407fd6e3ecea043b44580c229ccfa2a13f6d85866cf2b4a9628e
deleted: sha256:138c2e6554219de65614d88c15521bfb2da674cbb0bf840de161f89ff4264b96
deleted: sha256:ec353c2e1a673f456c4b78906d0d77f9d9456cfb5229b78c6a960bfb7496b76a
deleted: sha256:fe22765feaf3907526b4921c73ea6643ff9e334497c9b7e177972cf22f68ee93
deleted: sha256:ff845959c80148421a5c3ae11cc0e6c115f950c89bc949646be55ed18d6a2912
deleted: sha256:a4320831346648c03db64149eafc83092e2b34ab50ca6e8c13112388f25899a7
deleted: sha256:4c76020202ee1d9709e703b7c6de367b325139e74eebd6b55b30a63c196abaf3
deleted: sha256:d7afd92fb07236c8a2045715a86b7d5f0066cef025018cd3ca9a45498c51d1d6
deleted: sha256:9e63c5bce4585dd7038d830a1f1f4e44cb1a1515b00e620ac718e934b484c938
untagged: debian:jessie
untagged: debian@sha256:c1af755d300d0c65bb1194d24bce561d70c98a54fb5ce5b1693beb4f7988272f
deleted: sha256:7b0a06c805e8f23807fb8856621c60851727e85c7bcb751012c813f122734c8d
deleted: sha256:f96222d75c5563900bc4dd852179b720a0885de8f7a0619ba0ac76e92542bbc8

Total reclaimed space: 792.6 MB

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
foo                 latest              2f287ac753da        About a minute ago   3.98 MB
alpine              latest              88e169ea8f46        8 days ago           3.98 MB
busybox             latest              e02e811dd08f        2 months ago         1.09 MB
```

```bash
$ docker image prune --filter="label=deprecated"
$ docker image prune --filter="label=maintainer=john"
$ docker image prune --filter="label!=maintainer"
$ docker image prune --filter="label!=maintainer=john"
```
