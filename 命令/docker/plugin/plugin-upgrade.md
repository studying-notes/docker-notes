# `docker plugin upgrade` 命令

> 更新一个已存在的插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin upgrade [OPTIONS] PLUGIN [REMOTE]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--disable-content-trust` | `true` | 跳过镜像认证 |
| `--grant-all-permissions` | | 授予运行插件所需的所有权限 |
| `--skip-remote-check` | | 不检查指定的远程插件是否与镜像匹配 |

## 扩展信息

将现有插件升级到指定的远程插件。如果未指定任何远程，则 Docker 将重新拉取当前镜像并使用更新的版本。现有的所有对该插件的引用将继续起作用。在运行升级之前，必须禁用该插件。

## 示例

```bash
$ docker plugin install vieux/sshfs DEBUG=1

Plugin "vieux/sshfs:next" is requesting the following privileges:
 - network: [host]
 - device: [/dev/fuse]
 - capabilities: [CAP_SYS_ADMIN]
Do you grant the above permissions? [y/N] y
vieux/sshfs:next

$ docker volume create -d vieux/sshfs:next -o sshcmd=root@1.2.3.4:/tmp/shared -o password=XXX sshvolume

sshvolume

$ docker run -it -v sshvolume:/data alpine sh -c "touch /data/hello"

$ docker plugin disable -f vieux/sshfs:next

viex/sshfs:next

# Here docker volume ls doesn't show 'sshfsvolume', since the plugin is disabled
$ docker volume ls

DRIVER              VOLUME NAME

$ docker plugin upgrade vieux/sshfs:next vieux/sshfs:next

Plugin "vieux/sshfs:next" is requesting the following privileges:
 - network: [host]
 - device: [/dev/fuse]
 - capabilities: [CAP_SYS_ADMIN]
Do you grant the above permissions? [y/N] y
Upgrade plugin vieux/sshfs:next to vieux/sshfs:next

$ docker plugin enable vieux/sshfs:next

viex/sshfs:next

$ docker volume ls

DRIVER              VOLUME NAME
viuex/sshfs:next    sshvolume

$ docker run -it -v sshvolume:/data alpine sh -c "ls /data"

hello
```
