# `docker stack` 命令

> Manage Docker stacks

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker stack [OPTIONS] COMMAND
```

### 选项

| 名称 | 描述 |
| ------------- | ------------- |
| `--kubeconfig` | **Kubernetes** Kubernetes config file |
| `--orchestrator` | Orchestrator to use (swarm\|kubernetes\|all) |

### 命令

| 命令 | 描述 |
| ------------- | ------------- |
| [docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/) | Deploy a new stack or update an existing stack |
| [docker stack ls](https://docs.docker.com/engine/reference/commandline/stack_ls/) | List stacks |
| [docker stack ps](https://docs.docker.com/engine/reference/commandline/stack_ps/) | List the tasks in the stack |
| [docker stack rm](https://docs.docker.com/engine/reference/commandline/stack_rm/) | Remove one or more stacks |
| [docker stack services](https://docs.docker.com/engine/reference/commandline/stack_services/) | List the services in the stack |
