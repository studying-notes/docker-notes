# `docker system df` 命令

> 显示 Docker 磁盘使用情况

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker system df [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--verbose, -v` | 显示空间使用的详细信息 |


## 扩展信息

- 显示有关 Docker 守护进程使用的磁盘空间量的信息。

## 示例

```bash
$ docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              5                   2                   16.43 MB            11.63 MB (70%)
Containers          2                   0                   212 B               212 B (100%)
Local Volumes       2                   1                   36 B                0 B (0%)
```

```bash
$ docker system df -v

Images space usage:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE                SHARED SIZE         UNIQUE SIZE         CONTAINERS
my-curl             latest              b2789dd875bf        6 minutes ago       11 MB               11 MB               5 B                 0
my-jq               latest              ae67841be6d0        6 minutes ago       9.623 MB            8.991 MB            632.1 kB            0
<none>              <none>              a0971c4015c1        6 minutes ago       11 MB               11 MB               0 B                 0
alpine              latest              4e38e38c8ce0        9 weeks ago         4.799 MB            0 B                 4.799 MB            1
alpine              3.3                 47cf20d8c26c        9 weeks ago         4.797 MB            4.797 MB            0 B                 1

Containers space usage:

CONTAINER ID        IMAGE               COMMAND             LOCAL VOLUMES       SIZE                CREATED             STATUS                      NAMES
4a7f7eebae0f        alpine:latest       "sh"                1                   0 B                 16 minutes ago      Exited (0) 5 minutes ago    hopeful_yalow
f98f9c2aa1ea        alpine:3.3          "sh"                1                   212 B               16 minutes ago      Exited (0) 48 seconds ago   anon-vol

Local Volumes space usage:

NAME                                                               LINKS               SIZE
07c7bdf3e34ab76d921894c2b834f073721fccfbbcba792aa7648e3a7a664c2e   2                   36 B
my-named-vol                                                       0                   0 B
```

- `SHARED SIZE` 多个镜像共享的空间大小
- `UNIQUE SIZE` 指定镜像独占的空间大小
- `SIZE` 镜像的虚拟大小，是 `SHARED SIZE`、`UNIQUE SIZE` 之和
