# `docker node demote` 命令

> 从 Swarm 管理器降级一个或多个节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node demote NODE [NODE...]
```

## 扩展信息

- 降级现有的管理器，使其不再是管理器。这个命令的目标是 Swarm 中作为管理者的 docker engine。

## 示例

```bash
$ docker node demote <node name>
```
