# ` docker ps` 命令详解

> 列出容器

## 基本信息

### 用法

```
docker ps [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--all , -a` | 显示全部容器，默认只显示运行状态的容器 |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--last , -n` | 显示最新创建的 n 个的容器，状态不限，默认 `-1` 表示全部显示 |
| `--latest , -l` | 显示最新创建的那个容器，状态不限 |
| `--no-trunc` | 不截断输出，即完整显示 `CONTAINER ID` |
| `--quiet , -q` | 只显示 `CONTAINER ID` |
| `--size , -s` | 显示总文件大小 |

## 扩展信息

- `docker ps` 尽可能将暴露的端口归为一个范围。例如，暴露 `TCP` 端口 `100, 101, 102` 的容器在 `PORTS` 列中显示 `100-102/tcp`。

## 示例

### 不截断输出对比

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8c6d43312634        ubuntu              "bash"              9 seconds ago       Up 7 seconds                            demo

$ docker ps --no-trunc
CONTAINER ID                                                       IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8c6d43312634eafc48e880d6cf753b71a3a722bf37517d0f5fd001cd229a146c   ubuntu              "bash"              34 seconds ago      Up 32 seconds                           demo
```

### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

| 关键词 | 描述 |
| :-------------------- | :-------------------- |
| `id` | Container's ID |
| `name` | Container's name |
| `label` | An arbitrary string representing either a key or a key-value pair. Expressed as `<key>` or `<key>=<value>` |
| `exited` | An integer representing the container’s exit code. Only useful with `--all`. |
| `status` | One of `created`, `restarting`, `running`, `removing`, `paused`, `exited`, or `dead` |
| `ancestor` | Filters containers which share a given image as an ancestor. Expressed as `<image-name>[:<tag>]`, `<image id>`, or `<image@digest>` |
| `before` or `since` | Filters containers created before or after a given container ID or name |
| `volume` | Filters running containers which have mounted a given volume or bind mount. |
| `network` | Filters running containers connected to a given network. |
| `publish` or `expose` | Filters containers which publish or expose a given port. Expressed as `<port>[/<proto>]` or `<startport-endport>/[<proto>]` |
| `health` | Filters containers based on their healthcheck status. One of `starting`, `healthy`, `unhealthy` or `none`. |
| `isolation` | Windows daemon only. One of `default`, `process`, or `hyperv`. |
| `is-task` | Filters containers that are a “task” for a service. Boolean option (`true` or `false`) |

#### LABEL

`label` 过滤器仅根据单独的 `label` 或 `label` 和值来匹配容器。

```bash
$ docker ps --filter "label=color"

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
673394ef1d4c        busybox             "top"               47 seconds ago      Up 45 seconds                           nostalgic_shockley
d85756f57265        busybox             "top"               52 seconds ago      Up 51 seconds                           high_albattani
```

```bash
$ docker ps --filter "label=color=blue"

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
d85756f57265        busybox             "top"               About a minute ago   Up About a minute                       high_albattani
```

#### NAME

`name` 过滤器匹配容器名称的全部或部分。

```bash
$ docker ps --filter "name=nostalgic_stallman"

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
9b6247364a03        busybox             "top"               2 minutes ago       Up 2 minutes                            nostalgic_stallman
```

```bash
$ docker ps --filter "name=nostalgic"

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
715ebfcee040        busybox             "top"               3 seconds ago       Up 1 second                             i_am_nostalgic
9b6247364a03        busybox             "top"               7 minutes ago       Up 7 minutes                            nostalgic_stallman
673394ef1d4c        busybox             "top"               38 minutes ago      Up 38 minutes                           nostalgic_shockley
```

#### EXITED

`exited` 过滤器通过退出状态代码匹配容器。

```bash
$ docker ps -a --filter 'exited=0'

CONTAINER ID        IMAGE             COMMAND                CREATED             STATUS                   PORTS                      NAMES
ea09c3c82f6e        registry:latest   /srv/run.sh            2 weeks ago         Exited (0) 2 weeks ago   127.0.0.1:5000->5000/tcp   desperate_leakey
106ea823fe4e        fedora:latest     /bin/sh -c 'bash -l'   2 weeks ago         Exited (0) 2 weeks ago                              determined_albattani
48ee228c9464        fedora:20         bash                   2 weeks ago         Exited (0) 2 weeks ago                              tender_torvalds
```

找到退出状态为 `137` 的容器，这意味着它们是被 `SIGKILL(9)` 结束的。

```bash
$ docker ps -a --filter 'exited=137'

CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS                       PORTS               NAMES
b3e1c0ed5bfe        ubuntu:latest       "sleep 1000"           12 seconds ago      Exited (137) 5 seconds ago                       grave_kowalevski
a2eb5558d669        redis:latest        "/entrypoint.sh redi   2 hours ago         Exited (137) 2 hours ago                         sharp_lalande
```

下面的事件都会导致容器退出状态为 `137`：

- 容器 `init` 进程被手动杀死
- `docker kill` 杀死了容器
- Docker 守护进程重启过程中杀死了全部运行中的容器

#### STATUS

`status` 过滤器通过状态匹配容器，支持的状态有 `created`, `restarting`, `running`, `removing`, `paused`, `exited`, `dead`。

```bash
$ docker ps --filter status=running

CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS               NAMES
715ebfcee040        busybox                "top"               16 minutes ago      Up 16 minutes                           i_am_nostalgic
d5c976d3c462        busybox                "top"               23 minutes ago      Up 23 minutes                           top
9b6247364a03        busybox                "top"               24 minutes ago      Up 24 minutes                           nostalgic_stallman
```

```bash
$ docker ps --filter status=paused

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
673394ef1d4c        busybox             "top"               About an hour ago   Up About an hour (Paused)                       nostalgic_shockley
```

#### ANCESTOR

`ancestor` 过滤器根据容器的镜像或其后代匹配容器。过滤器支持以下镜像表示：

- `image` -> `image:latest`
- `image:tag`
- `image:tag@digest`
- `short-id`
- `full-id`

```bash
$ docker ps --filter ancestor=ubuntu

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
919e1179bdb8        ubuntu-c1           "top"               About a minute ago   Up About a minute                       admiring_lovelace
5d1e4a540723        ubuntu-c2           "top"               About a minute ago   Up About a minute                       admiring_sammet
82a598284012        ubuntu              "top"               3 minutes ago        Up 3 minutes                            sleepy_bose
bab2a34ba363        ubuntu              "top"               3 minutes ago        Up 3 minutes                            focused_yonath
```

```bash
$ docker ps --filter ancestor=ubuntu-c1

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
919e1179bdb8        ubuntu-c1           "top"               About a minute ago   Up About a minute                       admiring_lovelace
```

```bash
$ docker ps --filter ancestor=ubuntu:12.04.5

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
82a598284012        ubuntu:12.04.5      "top"               3 minutes ago        Up 3 minutes                            sleepy_bose
```

```bash
$ docker ps --filter ancestor=d0e008c6cf02

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
82a598284012        ubuntu:12.04.5      "top"               3 minutes ago        Up 3 minutes                            sleepy_bose
```

#### CREATE TIME

**before**

`before` 过滤器仅显示在给定 ID 或名称的容器之前创建的容器。

```bash
$ docker ps

CONTAINER ID        IMAGE       COMMAND       CREATED              STATUS              PORTS              NAMES
9c3527ed70ce        busybox     "top"         14 seconds ago       Up 15 seconds                          desperate_dubinsky
4aace5031105        busybox     "top"         48 seconds ago       Up 49 seconds                          focused_hamilton
6e63f6ff38b0        busybox     "top"         About a minute ago   Up About a minute                      distracted_fermat
```

```bash
$ docker ps -f before=9c3527ed70ce

CONTAINER ID        IMAGE       COMMAND       CREATED              STATUS              PORTS              NAMES
4aace5031105        busybox     "top"         About a minute ago   Up About a minute                      focused_hamilton
6e63f6ff38b0        busybox     "top"         About a minute ago   Up About a minute                      distracted_fermat
```

**since**

`since` 过滤器仅显示在给定 ID 或名称的容器之后创建的容器。

```bash
$ docker ps -f since=6e63f6ff38b0

CONTAINER ID        IMAGE       COMMAND       CREATED             STATUS              PORTS               NAMES
9c3527ed70ce        busybox     "top"         10 minutes ago      Up 10 minutes                           desperate_dubinsky
4aace5031105        busybox     "top"         10 minutes ago      Up 10 minutes                           focused_hamilton
```

#### VOLUME

`volume` 过滤器仅显示装载特定卷的容器或将卷装载在特定路径中的容器：

```bash
$ docker ps --filter volume=remote-volume --format "table {{.ID}}\t{{.Mounts}}"
CONTAINER ID        MOUNTS
9c3527ed70ce        remote-volume

$ docker ps --filter volume=/data --format "table {{.ID}}\t{{.Mounts}}"
CONTAINER ID        MOUNTS
9c3527ed70ce        remote-volume
```

#### NETWORK

`network` 过滤器仅显示连接到给定名称或 ID 的网络的容器。

```bash
$ docker run -d --net=net1 --name=test1 ubuntu top
$ docker run -d --net=net2 --name=test2 ubuntu top

$ docker ps --filter network=net1

CONTAINER ID        IMAGE       COMMAND       CREATED             STATUS              PORTS               NAMES
9d4893ed80fe        ubuntu      "top"         10 minutes ago      Up 10 minutes                           test1
```

```bash
$ docker network inspect --format "{{.ID}}" net1

8c0b4110ae930dbe26b258de9bc34a03f98056ed6f27f991d32919bfe401d7c5

$ docker ps --filter network=8c0b4110ae930dbe26b258de9bc34a03f98056ed6f27f991d32919bfe401d7c5

CONTAINER ID        IMAGE       COMMAND       CREATED             STATUS              PORTS               NAMES
9d4893ed80fe        ubuntu      "top"         10 minutes ago      Up 10 minutes                           test1
```

#### NETWORK

`publish` 和 `expose` 过滤器仅显示开放了给定端口号、端口范围、协议的容器，默认协议为 `TCP`。

```bash
$ docker run -d --publish=80 busybox top
$ docker run -d --expose=8080 busybox top

$ docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
9833437217a5        busybox             "top"               5 seconds ago       Up 4 seconds        8080/tcp                dreamy_mccarthy
fc7e477723b7        busybox             "top"               50 seconds ago      Up 50 seconds       0.0.0.0:32768->80/tcp   admiring_roentgen

$ docker ps --filter publish=80

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                   NAMES
fc7e477723b7        busybox             "top"               About a minute ago   Up About a minute   0.0.0.0:32768->80/tcp   admiring_roentgen
```

```bash
$ docker ps --filter expose=8000-8080/tcp

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
9833437217a5        busybox             "top"               21 seconds ago      Up 19 seconds       8080/tcp            dreamy_mccarthy
```

```bash
$ docker ps --filter publish=80/udp
```


### 根据关键词过滤

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Container ID |
| `.Image` | Image ID |
| `.Command` | Quoted command |
| `.CreatedAt` | Time when the container was created. |
| `.RunningFor` | Elapsed time since the container was started. |
| `.Ports` | Exposed ports. |
| `.Status` | Container status. |
| `.Size` | Container disk size. |
| `.Names` | Container names. |
| `.Labels` | All labels assigned to the container. |
| `.Label` | Value of a specific label for this container. For example `'{{.Label "com.docker.swarm.cpu"}}'` |
| `.Mounts` | Names of the volumes mounted in this container. |
| `.Networks` | Names of the networks attached to this container. |

当指定 `--format` 选项时，`ps` 命令将完全按照模板声明的方式输出数据，加上 `table` 参数时包含列标题，列项目通过 `:` 分隔。

```bash
$ docker ps --format "{{.ID}}: {{.Command}}"

a87ecb4f327c: /bin/sh -c #(nop) MA
01946d9d34d8: /bin/sh -c #(nop) MA
c1d3b0166030: /bin/sh -c yum -y up
41d50ecd2f57: /bin/sh -c #(nop) MA
```

```bash
$ docker ps --format "table {{.ID}}\t{{.Labels}}"

CONTAINER ID        LABELS
a87ecb4f327c        com.docker.swarm.node=ubuntu,com.docker.swarm.storage=ssd
01946d9d34d8
c1d3b0166030        com.docker.swarm.node=debian,com.docker.swarm.cpu=6
41d50ecd2f57        com.docker.swarm.node=fedora,com.docker.swarm.cpu=3,com.docker.swarm.storage=ssd
```
