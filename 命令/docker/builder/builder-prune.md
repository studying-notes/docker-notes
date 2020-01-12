# `docker builder prune` 命令

> 清除构建缓存文件

## 基本信息

### 描述

仅 [**API 1.39**](https://docs.docker.com/engine/api/v1.39/) 及以上版本支持这个命令。

### 用法

```
docker builder prune
```

### 选项

| 名称 | 描述 |
| ------------------------- | ------------------------------------------------------------ |
| `--all , -a` | Remove all unused images, not just dangling ones |
| `--filter` | Provide filter values (e.g. ‘unused-for=24h’) |
| `--force , -f` | Do not prompt for confirmation |
| `--keep-storage` | Amount of disk space to keep for cache |
