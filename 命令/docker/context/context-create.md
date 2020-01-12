# `docker context create` 命令

> 创建一个上下文

## 基本信息

### 用法

```
docker context create [OPTIONS] CONTEXT
```

### 选项

| 名称 | 描述 |
| ----------- | ----------- |
| `--default-stack-orchestrator` | 用于此上下文的堆栈操作的默认编排器 (swarm\|kubernetes\|all) |
| `--description` | 上下文的描述 |
| `--docker` | 设置 Docker 端点 |
| `--from` | 从一个命名上下文创建上下文 |
| `--kubernetes` | 设置 kubernetes 端点 |

## 扩展信息

- 创建新的上下文可以快速切换 CLI 配置以连接到不同的集群或单个节点。
- 为了从头创建上下文，必须提供 Docker 参数，如果有必要可以提供 kubernetes 参数。

## 示例

1. 创建带 Docker 端点和 kubernetes 端点的上下文

```bash
$ docker context create my-context \
      --docker host=/var/run/docker.sock \
      --kubernetes config-file=/home/me/my-kube-config
```

2. 从已存在的上下文创建新的上下文

```bash
$ docker context create my-context --from existing-context
```

3. 若不提供 `--from` 参数，则默认从当前上下文创建

```bash
$ docker context create my-context
```

4. 从现有的基于 `DOCKER_HOST` 的脚本中创建上下文

```bash
$ source my-setup-script.sh
$ docker context create my-context
```

5. 从已存在的上下文创建

```bash
$ docker context create my-context \
      --docker from=existing-context \
      --kubernetes config-file=/home/me/my-kube-config
```

```bash
$ docker context create my-context \
      --docker host=/var/run/docker.sock \
      --kubernetes from=existing-context
```
