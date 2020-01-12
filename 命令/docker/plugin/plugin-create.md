# `docker plugin create` 命令

> 从一个 rootfs 中创建一个插件及其配置。插件数据目录必须包含 config.json 和 rootfs 文件夹

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin create [OPTIONS] PLUGIN PLUGIN-DATA-DIR
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--compress` | 用 gzip 压缩上下文 |

## 扩展信息

- 在创建创建之前，必须准备好插件的根文件系统。

## 示例

```bash
$ ls -ls /home/pluginDir
total 4
4 -rw-r--r--  1 root root 431 Nov  7 01:40 config.json
0 drwxr-xr-x 19 root root 420 Nov  7 01:40 rootfs

$ docker plugin create plugin /home/pluginDir
plugin

$ docker plugin ls
ID                  NAME                TAG                 DESCRIPTION                  ENABLED
672d8144ec02        plugin              latest              A sample plugin for Docker   false
```
