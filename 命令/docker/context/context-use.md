# `docker context use` 命令

> 设置当前 Docker 的上下文

## 基本信息

### 用法

```
docker context use CONTEXT
```

### 扩展信息

- 当未设置 `DOCKER_HOST`、`DOCKER_CONTEXT` 环境变量以及 `--host`、`--context` 全局选项时，设置要使用的默认上下文。 要禁用上下文的使用，可以使用特殊的 `default` 上下文。
