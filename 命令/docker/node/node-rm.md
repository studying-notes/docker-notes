# `docker node rm` 命令详解

> 从 Swarm 管理器移除一个或多个节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node rm [OPTIONS] NODE [NODE...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 强制从 Swarm 移除一个节点 |

## 示例

### 移除一个已停止的节点

```bash
$ docker node rm swarm-node-02

Node swarm-node-02 removed from swarm
```

### 尝试移除一个正在运行的节点

```bash
$ docker node rm swarm-node-03

Error response from daemon: rpc error: code = 9 desc = node swarm-node-03 is not
down and can't be removed
```

正在运行的节点无法被移除。

### 强制移除一个无法访问的节点

如果失去对工作节点的访问权，或者需要关闭它，因为它已经被破坏或没有按照预期的方式运行，可以用 `--force` 选项。这可能会导致暂时性错误或中断，具体取决于节点上正在运行的任务的类型。

```bash
$ docker node rm --force swarm-node-03

Node swarm-node-03 removed from swarm
```

管理节点必须降级为工作节点 `docker node demote`，然后才能从 Swarm 中删除它。
