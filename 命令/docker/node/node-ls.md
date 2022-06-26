# `docker node ls` 命令详解

> 列出 Swarm 的节点

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node ls [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--quiet , -q` | 只显示 `NETWORK ID` |

## 扩展信息

- 列出 Docker Swarm 管理者知道的所有节点。

## 示例

### 列出全部节点

```bash
$ docker node ls

ID                           HOSTNAME        STATUS  AVAILABILITY  MANAGER STATUS
1bcef6utixb0l0ca7gxuivsj0    swarm-worker2   Ready   Active
38ciaotwjuritcdtn9npbnkuz    swarm-worker1   Ready   Active
e216jshn25ckzbvmwlnh5jr3g *  swarm-manager1  Ready   Active        Leader
```

> 上面的输出中隐藏了一列 `.Self` —— 指示该节点是否与当前 Docker 守护进程相同。`*` 表示这个节点是当前的 Docker 守护进程。

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f id=custom -f user=cxfans`。

目前支持的过滤关键词有：

- [id](#id) 节点的 ID。
- [label](#label) 适用于引擎标签，节点标签当前不用于过滤。
- [membership](#membership) 成员资格。
    - `accepted`
    - `pending`
- [name](#name) 节点的主机名。
- [role](#role) 角色。
    - `worker`
    - `manager`

#### ID

```bash
$ docker node ls -f id=1

ID                         HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
1bcef6utixb0l0ca7gxuivsj0  swarm-worker2  Ready   Active
```

#### LABEL

适用于引擎标签，节点标签当前不用于过滤。

```bash
$ docker node ls -f "label=foo"

ID                         HOSTNAME       STATUS  AVAILABILITY  MANAGER STATUS
1bcef6utixb0l0ca7gxuivsj0  swarm-worker2  Ready   Active
```

#### MEMBERSHIP

```bash
$ docker node ls -f "membership=accepted"

ID                           HOSTNAME        STATUS  AVAILABILITY  MANAGER STATUS
1bcef6utixb0l0ca7gxuivsj0    swarm-worker2   Ready   Active
38ciaotwjuritcdtn9npbnkuz    swarm-worker1   Ready   Active
```

#### NAME

```bash
$ docker node ls -f name=swarm-manager1

ID                           HOSTNAME        STATUS  AVAILABILITY  MANAGER STATUS
e216jshn25ckzbvmwlnh5jr3g *  swarm-manager1  Ready   Active        Leader
```

```bash
$ docker network ls --filter scope=local
NETWORK ID          NAME                DRIVER              SCOPE
e85227439ac7        bridge              bridge              local
0ca0e19443ed        host                host                local
ca13cc149a36        localnet            bridge              local
f9e115d2de35        none                null                local
```

#### ROLE

```bash
$ docker node ls -f "role=manager"

ID                           HOSTNAME        STATUS  AVAILABILITY  MANAGER STATUS
e216jshn25ckzbvmwlnh5jr3g *  swarm-manager1  Ready   Active        Leader
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Node ID |
| `.Self` | Node of the daemon (`true/false`, `true`indicates that the node is the same as current docker daemon) |
| `.Hostname` | Node hostname |
| `.Status` | Node status |
| `.Availability` | Node availability (“active”, “pause”, or “drain”) |
| `.ManagerStatus` | Manager status of the node |
| `.TLSStatus` | TLS status of the node (“Ready”, or “Needs Rotation” has TLS certificate signed by an old CA) |
| `.EngineVersion` | Engine version |

```bash
$ docker node ls --format "{{.ID}}: {{.Hostname}} {{.TLSStatus}}"
e216jshn25ckzbvmwlnh5jr3g: swarm-manager1 Ready
35o6tiywb700jesrt3dmllaza: swarm-worker1 Needs Rotation
```
