# `docker network disconnect` 命令详解

> 断开容器与网络的连接

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network disconnect [OPTIONS] NETWORK CONTAINER
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--force , -f` | 强制容器断开网络连接 |

## 扩展信息

- 容器必须正在运行才能将其与网络断开连接。

## 示例

```bash
$ docker network disconnect multi-host-network container1
```
