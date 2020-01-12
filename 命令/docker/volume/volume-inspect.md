# `docker volume create` 命令详解

> 显示一个或多个卷的详细信息

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--format, -f` | 用给定的 Go 模板格式化输出 |

## 示例

```bash
$ docker volume create
85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d
$ docker volume inspect 85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d
[
  {
      "Name": "85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d",
      "Driver": "local",
      "Mountpoint": "/var/lib/docker/volumes/85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d/_data",
      "Status": null
  }
]

$ docker volume inspect --format '{{ .Mountpoint }}' 85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d
/var/lib/docker/volumes/85bffb0677236974f93955d8ecc4df55ef5070117b0e53333cc1b443777be24d/_data
```
