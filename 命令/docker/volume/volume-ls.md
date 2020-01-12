# `docker volume ls` 命令详解

> 列出卷

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker volume ls [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--quiet , -q` | 只显示 `VOLUME NAME` |

## 扩展信息

## 示例

### 列出卷

```bash
$ docker volume create rosemary

rosemary

$ docker volume create tyler

tyler

$ docker volume ls

DRIVER              VOLUME NAME
local               rosemary
local               tyler
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f type=custom -f type=builtin`，同时返回自定义和内建网络。

目前支持的过滤关键词有：

- dangling (boolean - true or false, 0 or 1) 悬空镜像
- driver (a volume driver’s name) 驱动名
- label (`label=<key>` or `label=<key>=<value>`) 标签
- name (a volume’s name) 卷名

#### DANGLING

```bash
$ docker run -d  -v tyler:/tmpwork  busybox

f86a7dd02898067079c99ceacd810149060a70528eff3754d0b0f1a93bd0af18
$ docker volume ls -f dangling=true
DRIVER              VOLUME NAME
local               rosemary
```

#### DRIVER

```bash
$ docker volume ls -f driver=local

DRIVER              VOLUME NAME
local               rosemary
local               tyler
```

#### LABEL

```bash
$ docker volume create the-doctor --label is-timelord=yes

the-doctor
$ docker volume create daleks --label is-timelord=no

daleks
```

```bash
$ docker volume ls --filter label=is-timelord

DRIVER              VOLUME NAME
local               daleks
local               the-doctor
```

```bash
$ docker volume ls --filter label=is-timelord=yes

DRIVER              VOLUME NAME
local               the-doctor
```

```bash
$ docker volume ls --filter label=is-timelord=yes --filter label=is-timelord=no

DRIVER              VOLUME NAME
```

#### NAME

```bash
$ docker volume ls -f name=rose

DRIVER              VOLUME NAME
local               rosemary
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.Name` | Volume name |
| `.Driver` | Volume driver |
| `.Scope` | Volume scope (local, global) |
| `.Mountpoint` | The mount point of the volume on the host |
| `.Labels` | All labels assigned to the volume |
| `.Label` | Value of a specific label for this volume. For example `{{.Label "project.version"}}` |

```bash
$ docker volume ls --format "{{.Name}}: {{.Driver}}"

vol1: local
vol2: local
vol3: local
```
