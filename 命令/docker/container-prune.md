# `docker container prune` 命令详解

> 移除全部已停止的容器

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker container prune [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter` | 指定过滤条件 |
| `--force , -f` | 不提示是否确认，强制删除 |

## 示例

### 删除容器

```bash
$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
4a7f7eebae0f63178aff7eb0aa39cd3f0627a203ab2df258c1a00b456cf20063
f98f9c2aa1eaf727e4ec9c0283bc7d4aa4762fbdba7f26191f26c97f64090360

Total reclaimed space: 212 B
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

- until (`<timestamp>`) - only remove containers created before given timestamp
- label (`label=<key>`, `label=<key>=<value>`, `label!=<key>`, or `label!=<key>=<value>`) - only remove containers with (or without, in case `label!=...` is used) the specified labels.

```bash
$ docker ps -a --format 'table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.CreatedAt}}\t{{.Status}}'

CONTAINER ID        IMAGE               COMMAND             CREATED AT                      STATUS
61b9efa71024        busybox             "sh"                2017-01-04 13:23:33 -0800 PST   Exited (0) 41 seconds ago
53a9bc23a516        busybox             "sh"                2017-01-04 13:11:59 -0800 PST   Exited (0) 12 minutes ago

$ docker container prune --force --filter "until=5m"

Deleted Containers:
53a9bc23a5168b6caa2bfbefddf1b30f93c7ad57f3dec271fd32707497cb9369

Total reclaimed space: 25 B

$ docker ps -a --format 'table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.CreatedAt}}\t{{.Status}}'

CONTAINER ID        IMAGE               COMMAND             CREATED AT                      STATUS
61b9efa71024        busybox             "sh"                2017-01-04 13:23:33 -0800 PST   Exited (0) 44 seconds ago
```

```bash
$ docker ps -a --format 'table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.CreatedAt}}\t{{.Status}}'

CONTAINER ID        IMAGE               COMMAND             CREATED AT                      STATUS
53a9bc23a516        busybox             "sh"                2017-01-04 13:11:59 -0800 PST   Exited (0) 7 minutes ago
4a75091a6d61        busybox             "sh"                2017-01-04 13:09:53 -0800 PST   Exited (0) 9 minutes ago

$ docker container prune --force --filter "until=2017-01-04T13:10:00"

Deleted Containers:
4a75091a6d618526fcd8b33ccd6e5928ca2a64415466f768a6180004b0c72c6c

Total reclaimed space: 27 B

$ docker ps -a --format 'table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.CreatedAt}}\t{{.Status}}'

CONTAINER ID        IMAGE               COMMAND             CREATED AT                      STATUS
53a9bc23a516        busybox             "sh"                2017-01-04 13:11:59 -0800 PST   Exited (0) 9 minutes ago
```
