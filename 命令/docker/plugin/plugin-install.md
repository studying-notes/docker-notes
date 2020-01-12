# `docker plugin install` 命令

> 安装一个插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin install [OPTIONS] PLUGIN [KEY=VALUE...]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--alias` | | 插件别名 |
| `--disable` | | 安装时不启用插件 |
| `--disable-content-trust` | `true` | 跳过镜像认证 |
| `--grant-all-permissions` | | 授予运行插件所需的所有权限 |

## 扩展信息

- 安装插件并默认启用。Docker 首先在本地主机上寻找插件，如果该插件在本地不存在，则从 Registry 拉取。

## 示例

```bash
$ docker plugin install vieux/sshfs DEBUG=1

Plugin "vieux/sshfs" is requesting the following privileges:
 - network: [host]
 - device: [/dev/fuse]
 - capabilities: [CAP_SYS_ADMIN]
Do you grant the above permissions? [y/N] y
vieux/sshfs
```

```bash
$ docker plugin ls

ID                  NAME                  TAG                 DESCRIPTION                ENABLED
69553ca1d123        vieux/sshfs           latest              sshFS plugin for Docker    true
```
