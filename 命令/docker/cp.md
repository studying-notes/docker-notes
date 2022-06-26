# `docker cp` 命令详解

> 在容器和本地文件系统之间复制文件/文件夹

## 基本信息

### 用法

```
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--archive , -a` | Archive mode (copy all uid/gid information) |
| `--follow-link , -L` | Always follow symbol link in SRC_PATH |

## 扩展信息

- `docker cp` 复制 `SRC_PATH` 的内容到 `DEST_PATH` 中，可以从容器复制到本地文件系统，或者反过来。当为 `SRC_PATH` 和 `DEST_PATH` 中的一个指定了 `-` 时，还可以从 STDIN 流式传入或从 STDOUT 传出一个存档文件。复制时，容器可以是正在运行或者已经停止。
- `docker cp` 命令假定容器路径是相对于容器的根目录 `/`，这意味着路径开头的 `/` 是不起作用的，比如 `darwin:/root/Dockerfile` 和 `darwin:root/Dockerfile` 是一样的。本地主机路径即可以是当前工作目录的相对路径，也可以是绝对路径。
- `cp` 命令的行为类似 Unix 的 `cp -a` 命令，递归地复制文件夹、尽可能保留文件权限属性。所有权设置为目的地的用户和用户组。例如，复制一个由 root 用户创建的文件到容器，复制一个由执行 `docker cp` 命令的用户创建的文件到本地主机。但是，当指定了 `-a`，`docker cp` 将所有权设置为源的用户和用户组。
- 指定 `-L` 时，`docker cp` 会复制符号链接指向的内容，而不是复制符号链接，当内容不存在时，不会复制。默认情况下将复制符号链接而不是目标链接。

### `SRC_PATH` 和 `DEST_PATH`

假定路径分隔符是 `/`，参数 `SRC_PATH` 在前，`DEST_PATH` 在后，那么存在的情况如下：

- `SRC_PATH` 指定一个文件
    - `DEST_PATH` 不存在：文件将保存到以 `DEST_PATH` 命名的新文件中
    - `DEST_PATH` 不存在且是一个目录（`/` 结尾）：报错，目的文件夹必须存在
    - `DEST_PATH` 存在且是一个文件：内容覆盖
    - `DEST_PATH` 存在且是一个目录：文件将被复制到目录中，文件名不变
- `SRC_PATH` 指定一个文件夹
    - `DEST_PATH` 不存在：自动创建，然后复制源目录下的项目
    - `DEST_PATH` 存在且是一个文件：报错，不可以复制一个目录到文件
    - `DEST_PATH` 存在且是一个目录
        - `SRC_PATH` 不以 `/.` 结尾：复制源目录
        - `SRC_PATH` 以 `/.` 结尾：复制源目录下的项目

`:` 用于分隔容器标识和容器路径，但也可以用来分隔本地主机路径，比如 `file:name.txt`，但必须显式地表示，比如：

```
`/path/to/file:name.txt` or `./file:name.txt`
```

`docker cp` 无法复制某些系统文件，比如 `/proc`、`/sys`、`/dev`、tmpfs 中的资源，和容器中用户创建的挂载。但是仍可以手动在 `docker exec` 运行 `tar`。以下两个示例以不同的方式做了相同的事：

```bash
$ docker exec CONTAINER tar Ccf $(dirname SRC_PATH) - $(basename SRC_PATH) | tar Cxf DEST_PATH -
```

```bash
$ tar Ccf $(dirname SRC_PATH) - $(basename SRC_PATH) | docker exec -i CONTAINER tar Cxf DEST_PATH -
```
