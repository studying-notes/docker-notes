# `docker plugin ls` 命令

> 列出插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin ls [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--no-trunc` | 不截断输出，即完整显示 `ID` |
| `--quiet , -q` | 只显示 `ID` |

## 扩展信息

- 安装插件并默认启用。Docker 首先在本地主机上寻找插件，如果该插件在本地不存在，则从 Registry 拉取。

## 示例

```bash
$ docker plugin ls

ID                  NAME                             TAG                 DESCRIPTION                ENABLED
69553ca1d123        tiborvass/sample-volume-plugin   latest              A test plugin for Docker   true
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f id=custom -f user=cxfans`。

目前支持的过滤关键词有：

- enabled (boolean - true or false, 0 or 1)
- capability (string - currently `volumedriver`, `networkdriver`, `ipamdriver`, `logdriver`, `metricscollector`, or `authz`)

#### CAPABILITY

```bash
$ docker plugin install --disable vieux/sshfs

Installed plugin vieux/sshfs

$ docker plugin ls --filter enabled=true

NAME                  TAG                 DESCRIPTION                ENABLED
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Plugin ID |
| `.Name` | Plugin name |
| `.Description` | Plugin description |
| `.Enabled` | Whether plugin is enabled or not |
| `.PluginReference` | The reference used to push/pull from a registry |


```bash
$ docker plugin ls --format "{{.ID}}: {{.Name}}"

4be01827a72e: vieux/sshfs:latest
```
