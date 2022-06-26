# `docker config create` 命令

> 从文件或者 STDIN 创建一个配置

## 基本信息

### 描述

仅 [**API 1.30**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令可适用于 Swarm。

### 用法

```
docker config create [OPTIONS] CONFIG file|-
```

### 选项

| 名称 | 描述 |
| ------------------------- | ------------------------------------------------------------ |
| `--label , -l` | 配置标签 |
| `--template-driver` | [**API 1.37+**](https://docs.docker.com/engine/api/v1.37/) 模板驱动 |
