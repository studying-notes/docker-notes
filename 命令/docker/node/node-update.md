# `docker node update` 命令详解

> 更新有关节点的元数据，例如其可用性，标签或角色。

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node update [OPTIONS] NODE
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--availability` | 节点的可用性 (`active` \| `pause` \| `drain`) |
| `--label-add` | 添加或更新节点标签 (key=value) |
| `--label-rm` | 删除节点标签 |
| `--role` | 节点角色 (`worker` \| `manager`) |

## 示例

```bash
$ docker node update --label-add foo worker1
```

```bash
$ docker node update --label-add foo --label-add bar worker1
```

```bash
$ docker node update --label-add type=queue worker1
```
