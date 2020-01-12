# `docker network prune` 命令详解

> 移除全部未使用本地卷

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker volume prune [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter` | 指定过滤条件 |
| `--force , -f` | 不提示是否确认，强制删除 |

## 扩展信息

- 未使用的卷是指未被任何容器引用的卷。

## 示例

### 删除网络

```bash
$ docker volume prune

WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
07c7bdf3e34ab76d921894c2b834f073721fccfbbcba792aa7648e3a7a664c2e
my-named-vol

Total reclaimed space: 36 B
```
