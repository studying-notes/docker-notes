# `docker plugin disable` 命令

> 禁用一个插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin disable [OPTIONS] PLUGIN
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 强制禁用活动插件 |

## 示例

```bash
$ docker plugin ls

ID                  NAME                             TAG                 DESCRIPTION                ENABLED
69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   true
```

```bash
$ docker plugin disable tiborvass/sample-volume-plugin

tiborvass/sample-volume-plugin

$ docker plugin ls

ID                  NAME                             TAG                 DESCRIPTION                ENABLED
69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   false
```
