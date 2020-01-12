# `docker volume rm` 命令详解

> 移除一个或多个卷

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 不提示是否确认，强制删除 |

## 示例

```bash
$ docker volume rm hello
hello
```
