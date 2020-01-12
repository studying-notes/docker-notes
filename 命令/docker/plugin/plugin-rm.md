# `docker plugin rm` 命令

> 删除一个或多个插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin rm [OPTIONS] PLUGIN [PLUGIN...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 强制删除活动插件 |

## 示例

```bash
$ docker plugin disable tiborvass/sample-volume-plugin

tiborvass/sample-volume-plugin

$ docker plugin rm tiborvass/sample-volume-plugin:latest

tiborvass/sample-volume-plugin
```
