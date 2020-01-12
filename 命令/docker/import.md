# `docker import` 命令详解

> 从 tarball 文件导入内容以创建文件系统镜像

## 基本信息

### 用法

```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--change , -c` | Apply Dockerfile instruction to the created image |
| `--message , -m` | Set commit message for imported image |
| `--platform` | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.32+**](https://docs.docker.com/engine/api/v1.32/) Set platform if server is multi-platform capable |

## 扩展信息

- 既可以指定 `URL`，也可以指定  `-` 从 `STDIN` 获取数据。该 `URL` 可以指向包含文件系统的 `tar` 归档文件，也可以指向 `Docker` 主机上的单个文件。如果指定归档文件，Docker 会将其解压缩到相对于根目录 `/` 的容器中。 如果指定单个文件，则必须指定主机内的完整路径。
- `--change` 参数可以将 `Dockerfile` 的指令应用到已创建的镜像上，支持如下指令：`CMD` | `ENTRYPOINT` | `ENV` | `EXPOSE` | `ONBUILD` | `USER` | `VOLUME` | `WORKDIR`。

## 示例

### 远程导入创建镜像

```bash
$ docker import http://example.com/exampleimage.tgz
```

### 从本地导入创建镜像

1. 从 STDIN 通过管道导入

```bash
$ cat exampleimage.tgz | docker import - exampleimagelocal:new
```

2. 从 STDIN 通过管道导入且附加信息

```bash
$ cat exampleimage.tgz | docker import --message "New image imported from tarball" - exampleimagelocal:new
```

3. 从归档文件导入

```bash
$ docker import /path/to/exampleimage.tgz
```

### 从本地目录导入创建镜像

```bash
$ sudo tar -c . | docker import - exampleimagedir
```

### 从本地目录导入时修改配置

```bash
$ sudo tar -c . | docker import --change "ENV DEBUG true" - exampleimagedir
```

请注意示例中的 `sudo`，在用 `tar` 进行归档期间，必须保留文件的所有权(尤其是 root 的所有权)，非 root 权限执行所有权可能不会得到保留。
