# `docker images` 命令详解

> 镜像列表

## 基本信息

### 用法

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--all , -a` | 显示所有镜像（默认隐藏中间镜像） |
| `--digests` | 显示摘要 |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--no-trunc` | 不截断输出，即完整显示 `IMAGE ID` |
| `--quiet , -q` | 只显示 `IMAGE ID` |

## 扩展信息

- 默认的 `docker images` 将显示所有顶层镜像、存储库、标签以及大小。
- Docker 镜像具有中间层，可通过允许缓存每个步骤来提高可重用性，减少磁盘占用量、加快构建。 这些中间层默认情况下未显示。
- `SIZE` 是镜像及其所有父镜像占用的累积空间，这也是 `docker save` 镜像时创建的 tar 文件内容所占用的磁盘空间。
- 一个镜像如果有多个仓库名或标签会被多次列出。

## 示例

### 列出最近创建的镜像

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
demo                v3                  eef58dc87e2f        38 hours ago        64.2MB
demo                v2                  44ce0e2bb1f5        38 hours ago        64.2MB
demo                v1                  88aac8e89633        39 hours ago        64.2MB
alpine              latest              c85b8f829d1f        8 days ago          5.59MB
ubuntu              latest              549b9b86cb8d        9 days ago          64.2MB
fedora              latest              f0858ad3febd        2 months ago        194MB
centos              latest              0f3e07c0138f        2 months ago        220MB
```

### 根据名称和标签列出镜像

`docker images` 命令可选 `[REPOSITORY[:TAG]]` 参数，该参数将列表限制为与该参数匹配的镜像。如果指定 `REPOSITORY` 但没有 TAG，则 `docker images` 命令会列出给定存储库中的所有镜像。

```bash
$ docker images demo
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
demo                v3                  eef58dc87e2f        40 hours ago        64.2MB
demo                v2                  44ce0e2bb1f5        40 hours ago        64.2MB
demo                v1                  88aac8e89633        40 hours ago        64.2MB
```

```bash
$ docker images demo:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
demo                v1                  88aac8e89633        40 hours ago        64.2MB
```

### `IMAGE ID` 不截断输出

```bash
$ docker images --no-trunc demo:v1
REPOSITORY          TAG                 IMAGE ID                                                                  CREATED             SIZE
demo                v1                  sha256:88aac8e89633670b6acbe04de43c9f640179259823beacbd95e6450e55a0e253   40 hours ago        64.2MB
```

### 显示镜像摘要

```bash
$ docker images --digests centos
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
centos              latest              sha256:72942f390a71a1ecdebd4151c9bc4ff186fe8e480adcd7ddec84427118785482   0f3e07c0138f        2 months ago        220MB
```

### 根据关键词过滤显示

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。当前支持的过滤条件有：
- dangling (boolean - true or false)
- label (`label=<key>` or `label=<key>=<value>`)
- before (`<image-name>[:<tag>]`, `<image id>` or `<image@digest>`) - filter images created before given id or references
- since (`<image-name>[:<tag>]`, `<image id>` or `<image@digest>`) - filter images created since given id or references
- reference (pattern of an image reference) - filter images whose reference matches the specified pattern

#### 显示未打标签的镜像（悬空镜像）

```bash
$ docker images --filter "dangling=true"
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              8abc22fbb042        4 weeks ago         0 B
<none>              <none>              48e5f45168b9        4 weeks ago         2.489 MB
<none>              <none>              bf747efa0e2f        4 weeks ago         0 B
```

这将显示未标记的镜像，这些镜像是一棵镜像树的叶子，但不是中间层。当构建或者从 Registry 拉取新的镜像时将 `repo:tag` 标签从原镜像上移走，原镜像只能保留为 `<none>:<none>`，简而言之就是旧镜像的标识被新镜像拿走了，于是就会出现这些虚悬镜像。已经停止运行的虚悬镜像一般就失去价值了，可以全部清理掉：

```bash
$ docker rmi $(docker images -f "dangling=true" -q)
8abc22fbb042
48e5f45168b9
bf747efa0e2f
980fe10e5736
dea752e4e117
511136ea3c5a
```

#### 显示符合标签的镜像

标签过滤器基于单独的标签或标签和值的来匹配镜像。

1. 匹配标签，无所谓值

```bash
$ docker images --filter "label=com.example.version"
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
match-me-1          latest              eeae25ada2aa        About a minute ago   188.3 MB
match-me-2          latest              dea752e4e117        About a minute ago   188.3 MB
```

2. 匹配标签和值

```bash
$ docker images --filter "label=com.example.version=1.0"
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
match-me            latest              511136ea3c5a        About a minute ago   188.3 MB
```

#### 根据时间过滤镜像

假设现在有如下镜像：

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
```

1. 只显示在给定镜像之前创建的镜像

```bash
$ docker images --filter "before=image1"
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
image3              latest              511136ea3c5a        25 minutes ago       188.3 MB
```

2. 只显示在给定镜像之后创建的镜像

```bash
$ docker images --filter "since=image3"
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
image1              latest              eeae25ada2aa        4 minutes ago        188.3 MB
image2              latest              dea752e4e117        9 minutes ago        188.3 MB
```

#### 根据时间过滤镜像

`reference` 参数表示只显示引用 `[REPOSITORY[:TAG]]` 与指定模式匹配的镜像。

假设现在有如下镜像：

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              e02e811dd08f        5 weeks ago         1.09 MB
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             musl                733eb3059dce        5 weeks ago         1.21 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

1. 单模式匹配

```bash
$ docker images --filter=reference='busy*:*libc'
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

2. 多模式匹配

```bash
$ docker images --filter=reference='busy*:uclibc' --filter=reference='busy*:glibc'
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             uclibc              e02e811dd08f        5 weeks ago         1.09 MB
busybox             glibc               21c16b6787c6        5 weeks ago         4.19 MB
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Image ID |
| `.Repository` | Image repository |
| `.Tag` | Image tag |
| `.Digest` | Image digest |
| `.CreatedSince` | Elapsed time since the image was created |
| `.CreatedAt` | Time when the image was created |
| `.Size` | Image disk size |

当指定 `--format` 选项时，`history` 命令将完全按照模板声明的方式输出数据，加上 `table` 参数时包含列标题，列项目通过 `:` 分隔。

```bash
$ docker images --format "{{.ID}}: {{.Repository}}"
b6fa739cedf5: committ
30557a29d5ab: docker
746b819f315e: postgres
```

```bash
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY                TAG
b6fa739cedf5        committ                   latest
30557a29d5ab        docker                    latest
746b819f315e        postgres                  latest
```
