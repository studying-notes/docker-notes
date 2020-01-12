# `docker node promote` 命令详解

> 从 Swarm 管理器升级一个或多个节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node promote NODE [NODE...]
```

## 扩展信息

- 此命令只能在管理器节点上执行。

## 示例

```bash
$ docker node promote <node name>
```
