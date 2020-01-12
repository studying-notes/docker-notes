# ` docker start` 命令详解

> 启动一个或多个已停止的容器

## 基本信息

### 用法

```
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--attach , -a` | Attach STDOUT/STDERR and forward signals |
| `--checkpoint` | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) Restore from this checkpoint |
| `--checkpoint-dir` | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) Use a custom checkpoint storage directory |
| `--detach-keys` | Override the key sequence for detaching a container |
| `--interactive , -i` | Attach container’s STDIN |
