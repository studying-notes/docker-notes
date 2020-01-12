# `docker node` 命令

> 管理 Swarm 节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node COMMAND
```

### 命令

| 命令 | 描述 |
| ------------- | ------------- |
| [docker node demote](https://docs.docker.com/engine/reference/commandline/node_demote/) | 从 Swarm 管理器降级一个或多个节点 |
| [docker node inspect](https://docs.docker.com/engine/reference/commandline/node_inspect/) | 显示一个或多个节点的详细信息 |
| [docker node ls](https://docs.docker.com/engine/reference/commandline/node_ls/) | 列出 Swarm 的节点 |
| [docker node promote](https://docs.docker.com/engine/reference/commandline/node_promote/) | 从 Swarm 管理器升级一个或多个节点 |
| [docker node ps](https://docs.docker.com/engine/reference/commandline/node_ps/) | 列出在一个或多个节点上运行的任务，默认为当前节点 |
| [docker node rm](https://docs.docker.com/engine/reference/commandline/node_rm/) | 从 Swarm 管理器移除一个或多个节点 |
| [docker node update](https://docs.docker.com/engine/reference/commandline/node_update/) | 更新有关节点的元数据，例如其可用性，标签或角色。 |
