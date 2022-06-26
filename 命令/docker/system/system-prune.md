# `docker system prune` 命令详解

> 删除未使用的数据

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker system prune [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--all , -a` | 移除全部未使用镜像，默认只移除悬空镜像 |
| `--filter` | 指定过滤条件 |
| `--force , -f` | 不提示是否确认，强制删除 |
| `--volumes` | 移除数据卷 |

## 扩展信息

- 删除所有未使用的容器、网络、空悬镜像以及卷（可选）。

## 示例

### 删除数据

```bash
$ docker system prune

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] y

Deleted Containers:
f44f9b81948b3919590d5f79a680d8378f1139b41952e219830a33027c80c867
792776e68ac9d75bce4092bc1b5cc17b779bc926ab04f4185aec9bf1c0d4641f

Deleted Networks:
network1
network2

Deleted Images:
untagged: hello-world@sha256:f3b3b28a45160805bb16542c9531888519430e9e6d6ffc09d72261b0d26ff74f
deleted: sha256:1815c82652c03bfd8644afda26fb184f2ed891d921b20a0703b46768f9755c57
deleted: sha256:45761469c965421a92a69cc50e92c01e0cfa94fe026cdd1233445ea00e96289a

Total reclaimed space: 1.84kB
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

- until (`<timestamp>`) - only remove containers created before given timestamp
- label (`label=<key>`, `label=<key>=<value>`, `label!=<key>`, or `label!=<key>=<value>`) - only remove containers with (or without, in case `label!=...` is used) the specified labels.
