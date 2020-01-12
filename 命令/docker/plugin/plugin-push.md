# `docker plugin push` 命令

> 上传一个插件到 Registry

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin push [OPTIONS] PLUGIN[:TAG]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--disable-content-trust` | `true` | 跳过镜像认证 |

## 示例

```bash
$ docker plugin ls

ID                  NAME                  TAG                 DESCRIPTION                ENABLED
69553ca1d456        user/plugin           latest              A sample plugin for Docker false

$ docker plugin push user/plugin
```
