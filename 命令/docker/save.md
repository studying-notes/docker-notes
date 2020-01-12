# ` docker save` 命令详解

> 将一个或多个镜像保存到 tar 存档（默认情况下流式传输到 STDOUT）

## 基本信息

### 用法

```
docker save [OPTIONS] IMAGE [IMAGE...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--output , -o` | 写入指定文件 |

## 扩展信息

- 存档会包含全部的父镜像层和全部的标记和版本，或者指定的 `repo:tag`。

## 示例

### 创建一个可以被 `docker load` 的备份

```bash
$ docker save busybox > busybox.tar

$ ls -sh busybox.tar
2.7M busybox.tar

$ docker save --output busybox.tar busybox

$ ls -sh busybox.tar
2.7M busybox.tar

$ docker save -o fedora-all.tar fedora
$ docker save -o fedora-latest.tar fedora:latest
```

### 保存为 tar.gz 文件

```bash
$ docker save fans:latest | gzip > fans_latest.tar.gz
```

### 只保存特定标签的镜像

```bash
$ docker save -o ubuntu.tar ubuntu:love ubuntu:cxfans
```
