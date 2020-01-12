# `docker build` 命令详解

> 从 Dockerfile 构建镜像

## 基本信息

### 用法

```
docker build [OPTIONS] PATH | URL | -
```

### 选项

| 名称 | 默认 | 描述 |
| ------------------------- | ------- | ------------------------------------------------------------ |
| `--add-host` | | Add a custom host-to-IP mapping (host:ip) |
| `--build-arg` | | Set build-time variables |
| `--cache-from` | | Images to consider as cache sources |
| `--cgroup-parent` | | Optional parent cgroup for the container |
| `--compress` | | Compress the build context using gzip |
| `--cpu-period` | | Limit the CPU CFS (Completely Fair Scheduler) period |
| `--cpu-quota` | | Limit the CPU CFS (Completely Fair Scheduler) quota |
| `--cpu-shares , -c` | | CPU shares (relative weight) |
| `--cpuset-cpus` | | CPUs in which to allow execution (0-3, 0,1) |
| `--cpuset-mems` | | MEMs in which to allow execution (0-3, 0,1) |
| `--disable-content-trust` | `true` | Skip image verification |
| `--file , -f` | | Name of the Dockerfile (Default is ‘PATH/Dockerfile’) |
| `--force-rm` | | Always remove intermediate containers |
| `--iidfile` | | Write the image ID to the file |
| `--isolation` | | Container isolation technology |
| `--label` | | Set metadata for an image |
| `--memory , -m` | | Memory limit |
| `--memory-swap` | | Swap limit equal to memory plus swap: ‘-1’ to enable unlimited swap |
| `--network` | | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Set the networking mode for the RUN instructions during build |
| `--no-cache` | | Do not use cache when building the image |
| `--output , -o` | | [**API 1.40+**](https://docs.docker.com/engine/api/v1.40/) Output destination (format: type=local,dest=path) |
| `--platform` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.32+**](https://docs.docker.com/engine/api/v1.32/) Set platform if server is multi-platform capable |
| `--progress` | `auto` | Set type of progress output (auto, plain, tty). Use plain to show container output |
| `--pull` | | Always attempt to pull a newer version of the image |
| `--quiet , -q` | | Suppress the build output and print image ID on success |
| `--rm` | `true` | Remove intermediate containers after a successful build |
| `--secret` | | [**API 1.39+**](https://docs.docker.com/engine/api/v1.39/) Secret file to expose to the build (only if BuildKit enabled): id=mysecret,src=/local/secret |
| `--security-opt` | | Security options, only Windows |
| `--shm-size` | | Size of /dev/shm |
| `--squash` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Squash newly built layers into a single new layer |
| `--ssh` | | [**API 1.39+**](https://docs.docker.com/engine/api/v1.39/) SSH agent socket or keys to expose to the build (only if BuildKit enabled) (format: default\|[=\|[,]]) |
| `--stream` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.31+**](https://docs.docker.com/engine/api/v1.31/) Stream attaches to server to negotiate build context |
| `--tag , -t` | | Name and optionally a tag in the ‘name:tag’ format |
| `--target` | | Set the target build stage to build. |
| `--ulimit` | | Ulimit options |

## 扩展信息

`docker build` 命令从 Dockerfile 和“上下文”构建镜像。构建的上下文是位于指定路径或 URL 中的文件集合。构建过程可以引用上下文中的任何文件。

URL 参数可以是三种情况：Git 仓库、预打包 tarball 上下文和纯文本文件。

### Git 仓库

当 URL 参数指向 Git 仓库时，该仓库将作为构建上下文。系统以递归的方式（`git clone --recursive`）获取仓库及其子模块，不保留提交的历史记录。首先将仓库下载到本地主机的临时目录，然后将该目录作为上下文发送给 Docker 守护进程，因为客户端和守护进程不一定在同一个主机上。

Git URLs 允许将其部分片段作为构建上下文，以 `:` 分割表示。比如：

```bash
$ docker build https://github.com/docker/rootfs.git#container:docker
```

`https://github.com/docker/rootfs.git#container` 部分表示 Git 将 `check out` 的引用链接，可以是分支、标签或远程引用链接，`docker` 部分表示仓库中的子目录，该子目录将用作构建上下文。

下表列出了所有有效后缀及其构建上下文：

| 构建语法示例 | 提交的分支 | 上下文 |
| :----------------------------- | :-------------------- | :----------------- |
| `myrepo.git` | `refs/heads/master` | `/` |
| `myrepo.git#mytag` | `refs/tags/mytag` | `/` |
| `myrepo.git#mybranch` | `refs/heads/mybranch` | `/` |
| `myrepo.git#pull/42/head` | `refs/pull/42/head` | `/` |
| `myrepo.git#:myfolder` | `refs/heads/master` | `/myfolder` |
| `myrepo.git#master:myfolder` | `refs/heads/master` | `/myfolder` |
| `myrepo.git#mytag:myfolder` | `refs/tags/mytag` | `/myfolder` |
| `myrepo.git#mybranch:myfolder` | `refs/heads/mybranch` | `/myfolder` |

### tarball 上下文

当 URL 参数指向远程 tarball 文件时，这个 URL 会被发送给守护进程。

```bash
$ docker build http://server/context.tar.gz
```

下载操作将在运行 Docker 守护进程的主机上执行，该主机不一定与发出 `build` 命令的主机相同。Docker 守护进程将获取 `context.tar.gz` 文件，解压然后将其用作构建上下文。Tarball 上下文必须是符合标准 `tar` UNIX 格式的 `tar` 存档文件，可以是 `xz`、` bzip2`、` gzip` 或 ` identity`（无压缩）格式。

### 纯文本文件

不指定上下文，在 URL 中传入一个 Dockerfile 文件流，即该 URL 指向 Dockerfile 格式的纯文本文件，又或者通过管道从 STDIN 传入。比如：

```bash
$ docker build - < Dockerfile
```

在 Windows 下：

```powershell
Get-Content Dockerfile | docker build -
```

这种情况下，参数 `-f`、`--file` 将被忽略，系统会将传入内容放入名为 Dockerfile 的文件中，但是没有上下文。

默认情况下，`docker build` 命令将在构建上下文的根目录下寻找 `Dockerfile` 文件。用户可以指定 `-f`、`--file` 参数覆盖默认路径，该参数必须是指向构建上下文中的 `Dockerfile` 文件，若是一个相对路径，则解释为：当前目录为上下文的根目录，指定路径文件相对于当前目录。

在大多数情况下，最好将每个 `Dockerfile` 放在一个空目录中。然后，仅将构建 `Dockerfile` 所需的文件添加到该目录。为了提高构建的性能，还可以通过向该目录添加 `.dockerignore` 文件来排除文件和目录。

## 示例

### 将路径作为构建上下文

```bash
$ docker build .
```

示例中将当前工作目录 `.` 作为参数，前提是当前目录中存在 `Dockerfile` 文件。执行过程中，由于客户端不会对 `Dockerfile` 进行解析，当前目录中的所有文件都将被 `tar` 打包，然后发送到 Docker 守护进程，而守护进程可能在远程计算机上运行。在发送过程中，客户端会显示 `Sending build context` 信息。

为了在构建完成后保留中间容器文件，必须指定 `-rm=false`，这不会影响构建缓存。

### 将 URL 作为构建上下文

从 Git 仓库：

```bash
$ docker build github.com/creack/docker-firefox
```

从远程 Tarball 文件：

```bash
$ docker build -f ctx/Dockerfile http://server/ctx.tar.gz
```

`-f ctx/Dockerfile` 参数指定了 `ctx.tar.gz` 内部 `Dockerfile` 文件的路径，其中 `ADD` 本地文件的命令也同样必须是相对于 `ctx.tar.gz` 内部的根目录。

### 通过 `-` 构建

```bash
$ docker build - < Dockerfile
```

上面这种情况是没有上下文的，其中的 `ADD` 命令只能添加远程 URL 文件。

```bash
$ docker build - < context.tar.gz
```

这将从该压缩的上下文中构建镜像，格式可以是：`bzip2`、`gzip` 和 `xz`。

### .dockerignore 文件

```bash
$ echo ".git" > .dockerignore
```

### 给镜像打标签

```bash
$ docker build -t vieux/apache:2.0 .
```

构建镜像的同时打上标签。

```bash
$ docker build -t whenry/fedora-jboss:latest -t whenry/fedora-jboss:v2.1 .
```

也可以对同一个镜像打上多个标签。

### 指定 Dockerfile 文件

```bash
$ docker build -f Dockerfile.debug .
```

只需符合 `Dockerfile` 格式的文件即可。比如从 STDIN 读入也可以：

```bash
$ curl example.com/remote/Dockerfile | docker build -f - .
```

又或者同一个目录用不同配置的 Dockerfile 文件：

```bash
$ docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
$ docker build -f dockerfiles/Dockerfile.prod -t myapp_prod .
```

出于安全考虑，上下文仅限于当前目录及其子目录，`ADD ../file` 无法正确工作。

### 设置构建时临时变量

`ENV` 可以定义永久变量，`--build-arg` 可以设置构建时的临时变量。

```bash
$ docker build --build-arg HTTP_PROXY=http://10.20.30.2:1234 --build-arg FTP_PROXY=http://40.50.60.5:4567 .
```

这个参数在执行 `Dockerfile` 文件中的 `RUN` 命令时，可以像 `ENV` 一样访问变量，但是最终不会保留在镜像中。

```bash
$ export HTTP_PROXY=http://10.20.30.2:1234
$ docker build --build-arg HTTP_PROXY .
```

这和 `docker run -e` 工作过程相似。

### 指定容器的隔离技术

`--isolation=<value>` 参数可以指定容器的隔离技术。在 Linux 上，唯一支持 Linux 命名空间的默认选项。因此以下两个命令是等价的。

```bash
$ docker run -d busybox top
$ docker run -d --isolation default busybox top
```

在 Windows 上，可以指定以下值:

| 值 | 描述 |
| :-------- | :----------------------------------------------------------- |
| `default` | Use the value specified by the Docker daemon’s `--exec-opt` . If the `daemon` does not specify an isolation technology, Microsoft Windows uses `process` as its default value. |
| `process` | Namespace isolation only. |
| `hyperv` | Hyper-V hypervisor partition-based isolation. |

Windows Server 上的默认隔离技术是 `process`，而 Windows Client 上的默认隔离技术是 `hyperv`。在 Windows 10 1809 Client 上尝试 `--isolation process` 启动容器会失败。

```powershell
PS C:\> docker run -d microsoft/nanoserver powershell echo process
PS C:\> docker run -d --isolation default microsoft/nanoserver powershell echo process
PS C:\> docker run -d --isolation process microsoft/nanoserver powershell echo process
```

```powershell
PS C:\> docker run -d microsoft/nanoserver powershell echo hyperv
PS C:\> docker run -d --isolation default microsoft/nanoserver powershell echo hyperv
PS C:\> docker run -d --isolation hyperv microsoft/nanoserver powershell echo hyperv
```

### 添加新条目到容器的 `/etc/hosts`

```bash
$ docker build --add-host=docker:10.180.0.1 .
```

### 指定目标构建阶段

在拥有多个构建阶段的 `Dockerfile` 中，`--target` 可通过名称指定中间构建阶段作为某次构建的最终阶段，而该目标阶段之后的命令将被跳过。

```Dockerfile
FROM debian AS build-env
...

FROM alpine AS production-env
...
```

```bash
$ docker build -t mybuildimage --target build-env .
```

### 压缩镜像层（layers）

#### 概述

参数 `--squash` 是一个实验性功能，不稳定。

一旦镜像被构建，所有新层将被压缩为该新镜像的一个新层。压缩不会破坏任何现有的镜像，而是会创建具有压缩层内容的新镜像。这让 `Dockerfile` 中的命令看起来都是在一个层中创建的。这种方法会保留构建过程中产生的缓存文件。

当多层修改了同一文件时，比如一层增加文件，另一层删除了该文件，那么压缩层是有益的。 但对于其他用例，压缩层实际上可能会对性能产生负面影响； 提取由多层组成的镜像时，可以并行提取镜像层，允许在镜像之间共享镜像层以节省空间。

对于大多数用例，多阶段构建是更好的选择，因为它们对构建提供了更细粒度的控制，还可以利用构建器将来优化的优势。

#### 已知限制

- 当压缩镜像层时，生成的镜像不能利用与其他镜像共享的镜像层，可能会使用更多的空间。仍然支持共享基本镜像。

- 由于存储了两个镜像副本，可能会占用更多的空间，一个是用于所有缓存层的构建缓存，另一个用于压缩。

- 虽然压缩层可能会产生更小的镜像，但它可能会对性能产生负面影响，因为提取单个层需要更长的时间，而且下载单个层无法并行化。

- 当尝试压缩不更改文件系统的镜像时，压缩步骤将失败。

#### 前提条件

指定 `--experimental` 参数开启功能预览，或者在 `daemon.json` 文件中设置 `experimental: true`，然后重启 Docker 守护进程。

```json
{
 "experimental": true
}
```

查询是否已开启实验性功能：

```bash
$ docker version -f '{{.Server.Experimental}}'
true
```

#### 构建一个压缩镜像

**Dockerfile**

```Dockerfile
FROM busybox
RUN echo hello > /hello
RUN echo world >> /hello
RUN touch remove_me /remove_me
ENV HELLO world
RUN rm /remove_me
```

```bash
$ docker build --squash -t busybox:squash .

$ docker history busybox:squash

IMAGE CREATED CREATED BY SIZE COMMENT
4e10cb5b4cac 3 seconds ago 12 B merge sha256:88a7b0112a41826885df0e7072698006ee8f621c6ab99fca7fe9151d7b599702 to sha256:47bcc53f74dc94b1920f0b34f6036096526296767650f223433fe65c35f149eb
<missing> 5 minutes ago /bin/sh -c rm /remove_me 0 B
<missing> 5 minutes ago /bin/sh -c #(nop) ENV HELLO=world 0 B
<missing> 5 minutes ago /bin/sh -c touch remove_me /remove_me 0 B
<missing> 5 minutes ago /bin/sh -c echo world >> /hello 0 B
<missing> 6 minutes ago /bin/sh -c echo hello > /hello 0 B
<missing> 7 weeks ago /bin/sh -c #(nop) CMD ["sh"] 0 B
<missing> 7 weeks ago /bin/sh -c #(nop) ADD file:47ca6e777c36a4cfff 1.113 MB
```

Dockerfile 中每一个命令都会生成一个新镜像层，可以发现除了最新的一层，其他镜像层的名称都是 `<missing>`。
