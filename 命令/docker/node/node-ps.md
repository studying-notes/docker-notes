# `docker node ps` 命令详解

> 列出在一个或多个节点上运行的任务，默认为当前节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node ps [OPTIONS] [NODE...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--no-resolve` | 不将 ID 映射到名称 |
| `--no-trunc` | 不截断输出，即完整显示 `NETWORK ID` |
| `--quiet , -q` | 只显示 `TASK ID` |

## 示例

```bash
$ docker node ps swarm-manager1
NAME                                IMAGE        NODE            DESIRED STATE  CURRENT STATE
redis.1.7q92v0nr1hcgts2amcjyqg3pq   redis:3.0.6  swarm-manager1  Running        Running 5 hours
redis.6.b465edgho06e318egmgjbqo4o   redis:3.0.6  swarm-manager1  Running        Running 29 seconds
redis.7.bg8c07zzg87di2mufeq51a2qp   redis:3.0.6  swarm-manager1  Running        Running 5 seconds
redis.9.dkkual96p4bb3s6b10r7coxxt   redis:3.0.6  swarm-manager1  Running        Running 5 seconds
redis.10.0tgctg8h8cech4w0k0gwrmr23  redis:3.0.6  swarm-manager1  Running        Running 5 seconds
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f id=custom -f user=cxfans`。

目前支持的过滤关键词有：

- [name](#name) 任务名
- [id](#id) 任务 ID
- [label](#label)
- [desired-state](#desired-state)
    - `running`
    - `shutdown`
    - `accepted`

#### NAME

```bash
$ docker node ps -f name=redis swarm-manager1

NAME                                IMAGE        NODE            DESIRED STATE  CURRENT STATE
redis.1.7q92v0nr1hcgts2amcjyqg3pq   redis:3.0.6  swarm-manager1  Running        Running 5 hours
redis.6.b465edgho06e318egmgjbqo4o   redis:3.0.6  swarm-manager1  Running        Running 29 seconds
redis.7.bg8c07zzg87di2mufeq51a2qp   redis:3.0.6  swarm-manager1  Running        Running 5 seconds
redis.9.dkkual96p4bb3s6b10r7coxxt   redis:3.0.6  swarm-manager1  Running        Running 5 seconds
redis.10.0tgctg8h8cech4w0k0gwrmr23  redis:3.0.6  swarm-manager1  Running        Running 5 seconds
```

#### ID

```bash
$ docker node ps -f id=bg8c07zzg87di2mufeq51a2qp swarm-manager1

NAME                                IMAGE        NODE            DESIRED STATE  CURRENT STATE
redis.7.bg8c07zzg87di2mufeq51a2qp   redis:3.0.6  swarm-manager1  Running        Running 5 seconds
```

#### LABEL

```bash
$ docker node ps -f "label=usage"

NAME                               IMAGE        NODE            DESIRED STATE  CURRENT STATE
redis.6.b465edgho06e318egmgjbqo4o  redis:3.0.6  swarm-manager1  Running        Running 10 minutes
redis.7.bg8c07zzg87di2mufeq51a2qp  redis:3.0.6  swarm-manager1  Running        Running 9 minutes
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Task ID |
| `.Name` | Task name |
| `.Image` | Task image |
| `.Node` | Node ID |
| `.DesiredState` | Desired state of the task (`running`, `shutdown`, or `accepted`) |
| `.CurrentState` | Current state of the task |
| `.Error` | Error |
| `.Ports` | Task published ports |

```bash
$ docker node ps --format "{{.Name}}: {{.Image}}"
top.1: busybox
top.2: busybox
top.3: busybox
```
