# `docker load` 命令详解

> 从 tar 存档或标准输入文件加载镜像

## 基本信息

### 用法

```
docker load [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--input , -i` | Read from tar archive file, instead of STDIN |
| `--quiet , -q` | Suppress the load output |

## 扩展信息

- 同时还原镜像和标签。

## 示例

```bash
$ docker rm -f $(docker ps -aq)
$ docker rmi -f $(docker images -aq)
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

$ docker load < busybox.tar.gz
Loaded image: busybox:latest
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB

$ docker load --input fedora.tar
Loaded image: fedora:rawhide
Loaded image: fedora:20

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB
fedora              rawhide             0d20aec6529d        7 weeks ago         387 MB
fedora              20                  58394af37342        7 weeks ago         385.5 MB
fedora              heisenbug           58394af37342        7 weeks ago         385.5 MB
fedora              latest              58394af37342        7 weeks ago         385.5 MB
```
