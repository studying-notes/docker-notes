# `docker config inspect` 命令

> 显示一个或多个配置的详细信息

## 基本信息

### 描述

仅 [**API 1.30**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令可适用于 Swarm。

### 用法

```
docker config inspect [OPTIONS] CONFIG [CONFIG...]
```

### 选项

| 名称 | 描述 |
| --------------- | ------- |
| `--format , -f` | 用给定的 Go 模板格式化输出 |
| `--pretty` | 用友好可读的格式打印信息 |
