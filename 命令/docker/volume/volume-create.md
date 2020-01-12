# `docker volume create` 命令详解

> 创建卷

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker volume create [OPTIONS] [VOLUME]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--driver , -d` | `local` | 指定卷驱动程序名称 |
| `--label` | | 设置卷的元数据 |
| `--name` | | 指定卷名 |
| `--opt , -o` | | 设置驱动程序特定选项 |

## 扩展信息

- 创建一个供容器使用和存储数据的新卷。如果未指定名称，则 Docker 会生成一个随机名称。

## 示例

```bash
$ docker volume create hello
hello

$ docker run -d -v hello:/world busybox ls /world
```

- Docker 会在容器内的 `/world` 目录挂载卷，不存在该目录时会自动创建，但是挂载点必须是绝对路径，不支持相对路径。
- 多个容器可以同时使用一个卷。
- 卷的名称必须的唯一的，多次指定同一个卷不会报错、也不起任何作用。

### 特定于驱动程序的选项

- 某些卷驱动程序可能会使用一些选项来定制卷的创建。这些选项直接传递到卷驱动器。不同的卷驱动程序的选项可能会执行不同的操作或根本不执行任何操作。

```bash
$ docker volume create --driver fake \
    --opt tardis=blue \
    --opt timey=wimey \
    foo
```

- Windows 平台内置的 `local` 驱动不支持任何选项。
- Linux 平台内置的 `local` 驱动接受的选项类似 `mount` 命令，可以通过多个 `--opt` 提供多个参数。

```bash
$ docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    foo
```

```bash
$ docker volume create --driver local \
    --opt type=btrfs \
    --opt device=/dev/sda2 \
    foo
```

```bash
$ docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    foo
```
