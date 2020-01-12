# `docker network inspect` 命令详解

> 显示一个或多个网络的详细信息

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--format, -f` | 用给定的 Go 模板格式化输出 |
| `--verbose , -v` | 详细输出以进行诊断 |

## 扩展信息

- 默认情况下，该命令将所有结果呈现在 JSON 对象中。
