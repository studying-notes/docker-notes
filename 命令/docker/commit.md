# `docker commit` 命令详解

> 根据容器的更改创建新镜像

## 基本信息

### 用法

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

### 选项

| 名称 | 默认值 | 描述 |
| --------------- | ------- | --------------------------------------------------- |
| `--author , -a`  |        | 作者及其联系方式 |
| `--change , -c`  |        | 将 Dockerfile 指令应用于被创建的镜像 |
| `--message , -m` |        | 提交附加的信息 |
| `--pause , -p`   | `true` | 提交过程中暂停容器运行 |

## 扩展信息

将容器的文件、设置更改提交到新镜像中可能很有用，这样可以通过交互方式调试容器，或将工作数据导出到另一台服务器。但是，通常情况下，最好还是用 Dockerfile 以文档化和可维护的方式管理镜像。

提交操作不会包含容器内挂载卷中的任何数据。默认情况下，在提交镜像时，将暂停运行该容器及其进程，可以指定 `--pause=false` 让容器保持运行。

## 示例

### 提交一个容器

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4e1394d3b422        ubuntu              "bash"              22 hours ago        Up 22 hours                             demo

$ docker commit demo demo:v1
sha256:88aac8e89633670b6acbe04de43c9f640179259823beacbd95e6450e55a0e253

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
demo                v1                  88aac8e89633        17 seconds ago      64.2MB
```

### 提交带新配置的容器

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4e1394d3b422        ubuntu              "bash"              22 hours ago        Up 22 hours                             demo

$ docker inspect -f "{{ .Config.Env }}" demo
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]

$ docker commit --change "ENV DEBUG true" demo  demo:v2
sha256:44ce0e2bb1f53d5e6381f536f7498cd5126bbf0efac92b46679a3975c02d1b46

$ docker inspect -f "{{ .Config.Env }}" 44ce0e2bb
[PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin DEBUG=true]
```

### 提交带新指令的容器

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4e1394d3b422        ubuntu              "bash"              22 hours ago        Up 22 hours                             demo

$ docker commit --change='CMD ["top", "-b"]' -c "EXPOSE 80" demo  demo:v3
sha256:eef58dc87e2f8370d82cd51712c11eb4ae4c53246fb5dd4f8b3d4895dbe44381

$ docker run -d demo:v3
0be2ad17d1b8a11fe53cc5abb2c281c3798ec6729284d7d9d31c9fef8a797c43

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
0be2ad17d1b8        demo:v3             "top -b"            4 seconds ago       Up 2 seconds        80/tcp              pedantic_agnesi
```
