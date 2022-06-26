---
date: 2020-12-29T10:57:21+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker Compose v3.8 编排详解"  # 文章标题
# description: "文章描述"
url:  "posts/docker/build/compose"  # 设置网页永久链接
tags: [ "docker", "docker-build"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 指定构建 Dockerfile

```yaml
version: '3.1'
services:
    big_data_dev:
      build:
       context: ../
       dockerfile: DockerfileDev
      ports:
       - 81:80
      restart: unless-stopped
      container_name: big_data_dev_container
      image: big_data_dev_image
```

## 构建命令

在测试目录中，执行以下命令来启动应用程序：

```shell
docker-compose up
```

在后台执行该服务可以加上 `-d` 参数：

```shell
docker-compose up -d
```

## 指定文件

```shell
docker-compose -f docker/docker-compose.yml up
```

## docker-compose 与 dockerfile 的区别

**docker-compose** 是编排容器的。例如，有一个 php 镜像，一个 mysql 镜像，一个 nginx 镜像。如果没有 docker-compose，那么每次启动的时候，需要敲各个容器的启动参数，环境变量，容器命名，指定不同容器的链接参数等等一系列的操作，相当繁琐。而用了 docker-composer 之后，就可以把这些命令一次性写在 docker-composer.yml 文件中，以后每次启动这一整个环境（含 3 个容器）的时候，只要敲一个 docker-composer up 命令就行了。

**dockerfile** 的作用是从无到有的构建镜像。它包含安装运行所需的环境、程序代码等。这个创建过程就是使用 dockerfile 来完成的。Dockerfile — 为 docker build 命令准备的，用于建立一个独立的 image，在 docker-compose 里也可以用来实时构建。

docker-compose 在大型项目中需要配合很多镜像，这时候只需要写入 docker-compose.yml 中启动的时候 docker-compose up 管理多个 container，dockerfile 记录了一个镜像的制作过程。有些用了 dockerfile + docker-compose，是因为 docker-compose.yml 本身没有镜像构建的信息，如果镜像是从 docker registry 拉取下来的，那么 Dockerfile 就不需要；如果镜像是需要自己构建的，那就需要提供 Dockerfile。

## 编排文件结构与示例

```yaml
version: "3.7"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

在这个样本中，顶级结构由 `version`，`services`，`networks`，`volumes` 等等标签构成。

在 `services` 章节中，你可以定义若干个服务，每个服务通常运转着一个容器，这些服务构成了一个整体的设施栈，又或是服务群。

### [Dockerfile] 关于 `ENTRYPOINT` 和 `CMD`

在 Dockerfile 中讨论这三条命令：

首先，`RUN` 不考虑，其用途可以被视作执行一条 Shell 指令。

`CMD` 和 `ENTRYPOING` 相似但实际上区别较大：`CMD` 指定默认命令及其命令行参数、或者命令行参数的结尾的部分，并且能够在启动容器时被覆盖（通过外部命令行指定的方式），`ENTRYPOINT` 指定该容器被运行时的启动命令，可以附带命令行参数，这条命令在被实际执行时还会附加 `CMD` 所指定的内容作为其命令行的结尾部分。

#### 典型用法

所以一个惯用法是：

```dockerfile
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world"]
```

然后执行该容器的效果类似于：

```
$ docker run -it test-container
Hello world
$ docker run -it test-container David
Hello David
```

#### 用于你的容器

所以对于自定义容器来说，你的主要应用程序的路径可以被用作 `ENTRYPOINT` 而在 `CMD` 中提供默认参数：

```dockerfile
ENTRYPOINT ["/usr/local/app/my-app"]
CMD ["--help"]
```

这将会包装你的 my-app 像一个外露的工具，默认时显示其帮助屏，你可以指定参数去运行 my-app：

```shell
$ docker run -it my-app-container
[... help screen here ...]
$ docker run -it my-app-container server run --foreground
[ 等同于执行 my-app server run --foreground ]
```

END



### [Dockerfile] 多遍构建



多遍构建被典型地用于CI/CD。

例如步骤0可以被命名为 `builder`，负责从源码编译到目标文件，而步骤1则从步骤0中抽出目标文件用于部署打包，并生成最终的容器镜像，随后步骤0的中间层则被抛弃（仅指不被打包在结果容器中，实际上这些中间层在 Docker 的构建缓存中仍然是存在且可以被重用的），这些中间层不会出现在最终容器镜像中，从而有效地缩减了最终容器镜像的尺寸，而这个结果也是从语义上、从逻辑上被自洽的。

```dockerfile
FROM golang:1.7.3 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```

`COPY . .`: 将 `.` 项目中的当前目录复制到 `.` 镜像中的工作目录。

## 编排格式手册 - `service` 配置参考

> 接下来会是一个参考手册应有的章节结构，我们按照字母顺序列列举出了服务编排的指令，例如 `ports`，`volumes`，`cmd`，`entry` 等等。

Compose 文件是一个 YAML格式的文本文件，其中定义了 service、networks 以及 volumes。缺省时 docker-compose 使用和检索 `./docker-compose.yml` 文件并解释之。

> **Tip**: 你总是可以使用 `.yml` 或 `.yaml` 作为该脚本文件的后缀，它们都会正确工作。

service （服务）的配置包含若干定义，它们指明了如何将一个容器运行为服务，这些定义实际上会被传递给 `docker run` 作为其命令行参数的一部分。同样的道理，networks、volumes 等等的定义也采用同样的原理去影响诸如 `docker network create`，或者 `docker volume create` 等命令的实际运行。

你可以使用在配置定义值中使用环境变量，它们有类似于 BASH 变量替代的语法，你可以以 `${VARIABLE}`，请参阅 [变量替换](https://segmentfault.com/a/1190000023284156#变量替换) 小节的深入探讨。

接下来本章节中列举所有有效的服务配置项。

### build

该选项被用于构建，指定为构建镜像上下文路径。

`build` 可以是一个指向构建上下文的路径字符串，例如：

```yaml
version: "3.8"
services:
  webapp:
    build: ./dir
```

也可以是一个更详细的定义。这包括了 `context` 项所指定的路径，以及可选的 `dockerfile` 文件和构建参数 `args`：

```yaml
version: "3.8"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

如果在指定了 `build` 的同时还指定了 `image`，那么构建的容器名字会**被标记为该参数**，这就好像 `docker build -t container-name:tag dir` 做的那样：

```yaml
    build: "./dir"
    image: "company/webapp:v1.1.9"
```

> 对于 YAML 而言，避免歧义的安全做法是对字符串加上引号包围。

上面这个例子，会找到 `./dir` 文件夹中的构建上下文（默认是寻找到 `Dockerfile`）并完成构建，最后将其标记为 `company/webapp` 的名字，以及 `v1.1.9` 的 Tag。

> NOTE
>
> 当用在 docker stack 部署场景时：
>
> `build` 选项会被忽略。

### image

指定映像的名称。

```yaml
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```

如果映像在宿主机不存在，Compose 会尝试下拉它，除非你也指定了 `build` 项。

#### `context`

可以是一个包含 `Dockerfile` 的文件夹，也可以是一个指向 git repository 的 URL。

如果指定了一个相对路径，那么这个路径是相对于 `docker-compose.yml` 文件的。这个路径也会被传送给 Docker daemon 用于进行构建。

docker-compose 发动构建动作和标记构建结果（按照 `image` 名字），在那之后按照对应的名字使用它。

```
build:
  context: ./dir
```

#### `dockerfile`

可以指定不同于默认名称 `Dockerfile` 的其它文件名用于构建。注意同时必须指定路径到 `context`：

```
    build:
      context: .
      dockerfile: Dockerfile-alternate
```

#### `args`

指定构建参数。通常是指用于构建时的参数（参见 Dockerfile 中的 `ARG`）。

以下是一个简要的概述：

首先，在 Dockerfile 指定参数：

```dockerfile
ARG buildno
ARG gitcommithash

RUN echo "Build number: $buildno"
RUN echo "Based on commit: $gitcommithash"
```

然后指定构建参数的实际值（传入Map或者数组都是可以的）：

```
  build:
    context: .
    args:
      buildno: 1
      gitcommithash: cdc3b19
```

或：

```
build:
  context: .
  args:
    - buildno=1
    - gitcommithash=cdc3b19
```

> NOTE
>
> : 在 Dockerfile中，如果在
>
>
>
> ```
> FROM
> ```
>
>
>
> 之前指定
>
>
>
> ```
> ARG
> ```
>
> ，那么这个
>
>
>
> ```
> ARG
> ```
>
>
>
> 对于其后的
>
>
>
> ```
> FROM
> ```
>
>
>
> 闭包是无效的。
>
> 多个 `FROM` 分别切割出了多个构建的闭包。
>
> 想要 `ARG` 在每个 `FROM` 闭包中都有效，你需要在每个闭包中都指定它。
>
> 在 [Understand how ARGS and FROM interact](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) 中有更详细的相关讨论。

你可以略过显式指定构建参数。此时，该参数的实际值取决于构建时运行环境。

```
  args:
    - buildno
    - gitcommithash
```

> **NOTE**: YAML的布尔量（`true`, `false`, `yes`, `no`, `on`, `off`）必须用引号包围，以便 docker-compose正确处理。

#### `cache_from`

> since v3.2

指定一个映像列表，用于缓存的解决。

```
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

#### `labels`

> since v3.3

向构建的映像中添加元数据标签，可以是一个数组或一个字典。

我们推荐使用反向DNS标注性前缀以防止你的标签和使用者的标签相冲突：

```
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""

# anothor example
build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```

#### `network`

> Since v3.4

设置在 `RUN` 构建过程中要链接的网络，该网络也将被用于查询和提取依赖的容器。

```
build:
  context: .
  network: host
build:
  context: .
  network: custom_network_1
```

设为 `none` 则在构建时禁用网络查询和提取：

```
build:
  context: .
  network: none
```

#### shm_size

设置构建容器时的 `/dev/shm` 分区大小。整数格式按字节表示，但也可以使用字符串格式（[byte value](https://docs.docker.com/compose/compose-file/#specifying-byte-values)）：

```yaml
build:
  context: .
  shm_size: '2gb'

build:
  context: .
  shm_size: 10000000
```

#### `target`

> since v3.4

构建定义与 Dockerfile 中的特定的步骤（Stage），参阅 [multi-stage build docs](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)：

```
build:
  context: .
  target: prod
```

### `cap_add`, `cap_drop`

添加或者移除容器的 Linux 能力。完整的清单可以参阅 `man 7 capabilities`。

```
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

Linux 能力机制很大程度上是一种安全机制。具体含义、用途和引申属于 Linux 操作系统范畴，不再赘述。

### `cgroup_parent`

可选地为容器指定一个上级 `cgroup`。`cgroup` 也是 Linux 容器化实现的最重要的基本概念之一。

```
cgroup_parent: m-executor-abcd
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### `command`

覆盖容器内默认的 `command`.

```
command: bundle exec thin -p 3000
```

`command` 也可以被指定为一个列表。实际上这也是更被推荐的方式，无歧义而且安全，而且和 dockerfile 中的格式具有一致性：

```
command: ["bundle", "exec", "thin", "-p", "3000"]
```

### configs

为每个服务提供具体化的访问 `config` 的权限。

一个 `config` 包含一系列的配置信息，这些信息可能被通过多种途径所创建。容器的部署引用这些配置时，可以更好地区格诸如生产环境参数这样的问题。另一方面，敏感信息也因此可以被单独隔离到一个安全的区域中，在一定程度上减少了泄露的可能性。

> **NOTE**: 指定的配置必须已经存在，或者已经被定义在顶级 `configs` 中了（[defined in the top-level `configs` configuration](https://docs.docker.com/compose/compose-file/#configs-configuration-reference)）。否则整个容器栈的部署将会失败。

支持两个不同的语法变体格式。更详尽的信息应参考 [configs](https://docs.docker.com/engine/swarm/configs/)。

#### 短格式

只指定配置名。容器因此可以访问配置 `/<config_name` 和挂载它（挂载的源和目标均为该配置名）。

```
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

上面的例子使用短格式在 `redis` 容器的服务中定义了 `my_config` 和 `my_other_config` 的引用。这里的 `my_config` 指定为一个宿主机文件 `./my_config.txt`，而 `my_other_config` 被指定为外部的（资源），这意味着相应的资源已经在 Docker 中被定义了，或许是通过 `docker config create` 建立的，又或者是被别的容器栈部署所建立的。

如果外部资源找不到，那么容器栈的部署将会失败，并且抛出一个 `config not found` 的错误。

> **Note**: `config` 定义仅在 v3.3 及更高版本的 docker-compose 格式中被支持。

#### 长格式

长格式提供更多的信息来表述一个 `config` 在哪儿，如何被找到，怎么被使用：

- `source`: 配置名

- `target`: 该配置将被挂载到容器中的路径。默认为 `/<source>`

- `uid` & `gid`: 数字值的 Linux/Unix `UID` 和 `GID`，如果没有指定则为0。Windows中不支持。

- ```
  mode
  ```

  : 8进制的文件权限。默认值为



  ```
  0444
  ```

  。

  配置是不可写的，因为它们被挂载于临时的文件系统中。因此如果你设置了写许可，这将被忽略。

  可执行位是可以被设置的。

下面的例子类似于短格式的例子：

```
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

在这里，`redis` 容器的服务并未访问 `my_other_config`。

你可以授权一个服务访问多个配置，你也可以混用长短格式。

（在顶级）定义一个配置（`config`）并未隐含着某个服务就能访问到它。

### `container_name`

指定一个自定义的容器名，而不是由 docker-compose 自己生成一个默认的。

```
container_name: my-web-container
```

由于 Docker 容器名必须是唯一的，所以你无法伸缩一个自定义了容器名的服务。

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### `credential_spec`

> since v3.3
>
> 从 v3.8 开始支持被用于组管理服务账户 gMSA（group Managed Service Account）方式。

为受控服务账户配置凭据。这个选项只被用于 Windows 容器服务。`credential_spce` 只能使用格式 `file://<filename>` or `registry://<value-name>`。

当使用 `file:` 时，被参考的文件必须被置于 Docker 数据文件夹（通常是 `C:\ProgramData\Docker\`）的 `CredentialSpec` 子目录之下。下面的例子将会从 `C:\ProgramData\Docker\CredentialSpecs\my-credential-sp` 载入凭据信息：

```
credential_spec:
  file: my-credential-spec.json
```

当使用 `registry:` 时，凭据信息将会被从 Docker daemon 主机的 Windows Registry 中读入。一个注册表表项必须位于：

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\CredentialSpecs
```

之中。

下面的例子读入 `my-credential-spec` 注册表项值：

```
credential_spec:
  registry: my-credential-spec
```

#### gMSA 配置例子

当为一个服务配置 gMSA 凭据时，参考如下的例子：

```
version: "3.8"
services:
  myservice:
    image: myimage:latest
    credential_spec:
      config: my_credential_spec

configs:
  my_credentials_spec:
    file: ./my-credential-spec.json|
```

### depends_on

表示服务之间的依赖关系。服务依赖引发如下的行为：

- `docker-compose up` 按照依赖顺序依次启动服务。在下面的例子中，`db` 和 `redis` 先于 `web` 被启动。
- `docker-compose up SERVICE` 自动包括了 `SERVICE` 的依赖项。在下面的例子中，`docker-compose up web` 将会自动启动 `db` 和 `redis`。
- `docker-compose stop` 按照依赖顺序依次停止服务。在下面的例子中，`web` 将会被先于 `db` 和 `redis` 被停止。

简单的示例如下：

```yaml
version: "3.8"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> 使用 `depends_on` 时应该注意的几件事：
>
> - `depends_on` 并不意味着等待 `db` 和 `redis` 就绪后才启动 `web`，而是在它们被启动后就会接着启动 `web`。如果要想等到服务就绪可用，应该参阅 [Controlling startup order](https://docs.docker.com/compose/startup-order/)。

### `deploy`

> **[Version 3](https://docs.docker.com/compose/compose-file/compose-versioning/#version-3) only.**

指定和部署以及运行相关的配置。

只会对部署到一个使用 [docker stack deploy](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 的 [swarm](https://docs.docker.com/engine/swarm/) 有影响。

在 `docker-compose up` 和 `docker-compose run` 时被忽略。

```
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

几个子选项是可用的：

#### `endpoint_mode`

swarm.

> Since **[Version 3.2](https://docs.docker.com/compose/compose-file/compose-versioning/#version-3).**

指定外部客户端连接到一个 swarm 集群时使用的服务发现方法。

- ```
  endpoint_mode: vip
  ```



  \- Docker 为服务请求一个虚拟IP（

  ```
  VIP
  ```

  ）用于访问。

  Docker在客户端和服务的有效的工作节点之间对请求进行自动负载均衡路由。客户端无需知道服务有多少可用节点，也不必知道服务节点的IP地址和端口号。

  这是默认方式。

- `endpoint_mode: dnsrr` - 使用 DNS round-robin (DNSRR) 算法进行服务发现。Docker会为服务设置一个DNS条目，因而在进行对应的 DNS 解析时通过服务名称会返回一个IP地址清单。客户端因此直接选择一个具体的端点进行访问。

```yaml
version: "3.8"

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```

`endpoint_mode` 的选项也被同样地用作 swarm mode 命令行选项（参阅 [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/)）。至于 docker swarm命令的一个快捷清单，可以查阅 [Swarm mode CLI commands](https://docs.docker.com/engine/swarm/#swarm-mode-key-concepts-and-tutorial)。

要学习更多关于 swarm mode 的网络模型已经服务发现机制 的知识，请查看 [Configure service discovery](https://docs.docker.com/engine/swarm/networking/#configure-service-discovery)。

#### `labels`

为服务指定标签。这些标签只被作用于对应的服务，而不是被应用到服务的容器或者容器实例上。

```
version: "3.8"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

要为容器设置标签的话，在 `deploy` 之外为服务指定 `labels`：

```
version: "3.8"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

#### `mode`

可以是 `global` 或 `replicated`。`global` 表示严格地一个 swarm node 跑一个服务，`replicated` 表示可以跑多个容器实例。默认是 `replicated`。

参阅 [swarm](https://docs.docker.com/engine/swarm/) 主题下的 [Replicated and global services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)。

```
version: "3.8"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

#### `placement`

指定 constaints 和 preferences。

参阅docker服务建立的相关文档以了解更多的关于 [constraints](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint) 和 [preferences 的相关信息，包括相应的语法，可用的类型等等的完整描述。

```
version: "3.8"
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```

#### `max_replicas_per_node`

> Since [version 3.8](https://docs.docker.com/compose/compose-file/compose-versioning/#version-38).

如果一个服务是可副本的 `replicated` (这是默认的), max_replicas_per_node 将会 [限制副本数量（limit the number of replicas）](https://docs.docker.com/engine/reference/commandline/service_create/#specify-maximum-replicas-per-node---replicas-max-per-node) 。

当太多任务申请新的任务节点且超出了 max_replicas_per_node 限制值时，一个 `no suitable node (max replicas per node limit exceed)` 错误将会被抛出。

```
version: "3.8"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
      placement:
        max_replicas_per_node: 1
```

#### `replicas`

如果服务是 `replicated` 的，`replicas` 则为其指定一个数值，此数值指示了一个 swarm 节点上最多可以跑多少个容器实例。

```
version: "3.7"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

#### `resources`

配置资源约束。

> NOTE
>
> : 对于非 swarm mode 而言，这个表项替换
>
>
>
> older resource constraint options
>
> （诸如
>
>
>
> ```
> cpu_shares
> ```
>
> ,
>
>
>
> ```
> cpu_quota
> ```
>
> ,
>
>
>
> ```
> cpuset
> ```
>
> ,
>
>
>
> ```
> mem_limit
> ```
>
> ,
>
>
>
> ```
> memswap_limit
> ```
>
> ,
>
>
>
> ```
> mem_swappiness
> ```
>
>
>
> 等在版本3之前版本中的表项）。
>
> 在 [Upgrading version 2.x to 3.x](https://docs.docker.com/compose/compose-file/compose-versioning/#upgrading) 中有相应的描述。

这些资源约束表项都具有一个单一值，相当于 `docker service create` 中的等价物。

在如下的例子中，`redis` 服务被约束为不可使用超出50M的内存，单核50%的CPU使用率，同时也保留 20M 内存以及 25%的CPU使用率作为基准值。

```
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

以下的主题描述 swarm 场景下的服务或容器资源约束的可用选项。

##### Out Of Memory Exceptions (OOME)

企图在你的服务和容器实例中使用超过系统拥有的内存，那么将得到 Out Of Memory Exception (OOME) 。此时，容器实例，或者 Docker daemon，可能会被内核的 OOM 管理器所清除。

要防止这样的情况发生，请确定你的应用程序合法有效地使用内存。对于这样的风险，查阅 [Understand the risks of running out of memory](https://docs.docker.com/engine/admin/resource_constraints/#understand-the-risks-of-running-out-of-memory) 以获知进一步的评估须知。

#### `restart_policy`

指示当容器实例退出时，如何重启。替换 `restart`：

- `condition`: 可以为 `none`, `on-failure` 或 `any` (默认为 `any`)
- `delay`: 在尝试重启之前的等候时长（默认为0）。应该为其指定一个 [duration](https://docs.docker.com/compose/compose-file/#specifying-durations)。
- `max_attempts`: 试图尝试重启多少次后放弃重启的尝试。默认为不放弃。
- `window`: 要确定一次重启是否成功，需要等候的时长。默认为无等待立即认定为已成功。应该为其指定一个 [duration](https://docs.docker.com/compose/compose-file/#specifying-durations)。

```
version: "3.8"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

#### `rollback_config`

> [Version 3.7 file format](https://docs.docker.com/compose/compose-file/compose-versioning/#version-37) and up

在滚动更新失败的场景下服务应该如何回滚：

- `parallelism`：同时回滚的容器的数量值。如果设置为0，所有容器将被同时回滚。
- `delay`: 每个容器组被回滚前的等待时长（默认为0）
- `failure_action`: 一个回滚失败时应当执行的动作。可以是 `continue` 或 `pause`（默认为`pause`）
- `monitor`: 失败的回滚状态被更新到监视器的周期（`ns|us|ms|s|m|h`）默认为 `0s`。
- `max_failure_ratio`: 回滚时失败的可容忍的比例（默认为0）
- `order`: 回滚的操作顺序。可以为 `stop-first` 或 `start-first`（默认为 `stop-first`）

#### `update_config`

指示服务应该如何被更新。这对于配置为滚动更新时有用：

- `parallelism`：同时更新的容器的数量值。如果设置为0，所有容器将被同时回滚。
- `delay`: 每个容器组被更新前的等待时长（默认为0）
- `failure_action`: 一个更新失败时应当执行的动作。可以是 `continue`, `rollback` 或 `pause`（默认为`pause`）
- `monitor`: 失败的更新状态被更新到监视器的周期（`ns|us|ms|s|m|h`）默认为 `0s`。
- `max_failure_ratio`: 更新时失败的可容忍的比例（默认为0）
- `order`: 更新的操作顺序。可以为 `stop-first` 或 `start-first`（默认为 `stop-first`）

> **NOTE**：`order` 只在 v3.4 及之后有效。

```
version: "3.8"
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```

#### `DOCKER STACK DEPLOY` 不支持者

下列的子选项（为 `docker-compose up` 和 `docker-compose run` 所支持）是在 `docker stack deploy` 中不被支持的：

- [build](https://docs.docker.com/compose/compose-file/#build)
- [cgroup_parent](https://docs.docker.com/compose/compose-file/#cgroup_parent)
- [container_name](https://docs.docker.com/compose/compose-file/#container_name)
- [devices](https://docs.docker.com/compose/compose-file/#devices)
- [tmpfs](https://docs.docker.com/compose/compose-file/#tmpfs)
- [external_links](https://docs.docker.com/compose/compose-file/#external_links)
- [links](https://docs.docker.com/compose/compose-file/#links)
- [network_mode](https://docs.docker.com/compose/compose-file/#network_mode)
- [restart](https://docs.docker.com/compose/compose-file/#restart)
- [security_opt](https://docs.docker.com/compose/compose-file/#security_opt)
- [userns_mode](https://docs.docker.com/compose/compose-file/#userns_mode)

> **Tip:** 请参阅 [如何为 service 服务模式和 swarm 集群模式使用卷而配置 docker-stack.yml 文件 - how to configure volumes for services, swarms, and docker-stack.yml files](https://docs.docker.com/compose/compose-file/#volumes-for-services-swarms-and-stack-files) 章节。 Volumes（卷）在 swarms 模式和 services 模式中*是*被支持的，但你只能采用命名卷，又或是与受约束于有权访问*必需*卷的节点的服务关联.

### `devices`

要被映射的设备清单。其用法和 docker 命令的 `--device` 相同。

```
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### `dns`

自定义 DNS 服务器列表。可以指定单一值或一个清单。

```
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

### `dns_search`

自定义DNS搜索域名。可以指定单一值或一个清单。

```
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

### `entrypoint`

覆盖 dockerfile 中定义的默认的 entrypoint 值。

```
entrypoint: /code/entrypoint.sh
```

入口点也可以是一个清单：

```
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

> **NOTE**: 设置一个 `entrypoint` 不但覆盖 Dockerfile 中的任何 `ENTRYPOINT` 默认值，还会清理 Dockerfile 中的任何 `CMD` 默认值。所以 Dockerfile 中的 `CMD` 将会被忽略。

### `env_file`

从给定的文件中引入环境变量值。可以是单一值或一个清单。

```
env_file: .env
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

对于 `docker-compose -f FILE` 来说，`env_file` 的路径是相对于 `FILE` 所在文件夹的。

在 `environment` 中声明的环境变量将会覆盖掉这里所引入的值。

对应的文件中，每一行应该使用 `VAR=VAL` 格式定义一个环境变量。行首为 `#` 表示为注释行，和空白行一样被忽略。

```
# Set Rails/Rack environment
RACK_ENV=development
```

> **NOTE**: 如果服务定义了 `build` 项，在构建过程中，由 `env_file` 所定义的环境变量并不可见。只能使用 `build` 的子选项 `args` 去定义构建时环境变量值。

`VAL` 的值被原样照用，且不能被修改。例如如果值由引号所包围，那么值的表示量中也包含引号。

*环境变量文件的顺序也需要被注意*。位置靠后的环境变量文件中所定义的变量值会覆盖掉早前定义的旧值。

> 按
>
> ：原文竟然说了这么多！
>
> Keep in mind that *the order of files in the list is significant in determining the value assigned to a variable that shows up more than once*. The files in the list are processed from the top down. For the same variable specified in file `a.env` and assigned a different value in file `b.env`, if `b.env` is listed below (after), then the value from `b.env` stands. For example, given the following declaration in `docker-compose.yml`:

```
services:
  some-service:
    env_file:
      - a.env
      - b.env
```

And the following files:

```
# a.env
VAR=1
```

and

```
# b.env
VAR=hello
```

`$VAR` is `hello`.

### environment

添加环境变量。可以使用一个数组或者一个字典。任何布尔量：true, false, yes, no 等等都必须用引号包围为字符串字面量。

只有 key 值的环境变量的 value 值依赖于 docker-compose 运行时的主机环境，这对于防止敏感信息泄露是有用的。

```yaml
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

# 或
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

> **NOTE**: 如果服务定义了 `build` 项，在构建过程中，由 `env_file` 所定义的环境变量并不可见。只能使用 `build` 的子选项 `args` 去定义构建时环境变量值。

### `expose`

暴露端口到链接的服务。这些端口并不会被发布到宿主机。只能指定内部端口被用于暴露。

```
expose:
 - "3000"
 - "8000"
```

### `external_links`

将在 `docker-compose.yml` 之外启动的容器链接到给定服务上。

和遗留选项 `links` 有相似的语义。

```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

> Note
>
> 在 docker-compose.yml 之外建立的容器（The externally-created containers）必须至少连接到一个相同的网络，才能与一个定义在 docker-compose.yml 中的服务相链接 。[Links](https://docs.docker.com/compose/compose-file/compose-file-v2#links) 选项已经过时了，我们推荐使用 [networks](https://docs.docker.com/compose/compose-file/#networks) 来代替它。
>
> **NOTE**: 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

更推荐的做法是通过 `networks` 构造一个子网以进行容器之间的链接。

### `extra_hosts`

添加主机名映射。这些映射关系会被添加到 `/etc/hosts` 中。此功能等价于命令行参数 `--add-host`。

```
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

对于这个服务来说，在容器中的 `/etc/hosts` 文件中，相应的主机名及其IP将被建立为一个条目。例如：

```
162.242.195.82  somehost
50.31.209.229   otherhost
```

### `healthcheck`

> since v2.1

用于确认一个服务是否是“健康”的。参阅 [HEALTHCHECK Dockerfile instruction](https://docs.docker.com/engine/reference/builder/#healthcheck)。

```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

`interval`, `timeout` 和 `start_period` 应该被指定为 [durations](https://docs.docker.com/compose/compose-file/#specifying-durations).

> **Note**: `start_period` 只在 v3.4 及以后可用。

`test` 必须是一个单一字符串值或一个列表。如果是一个列表，那么第一项必须是 `NONE`, `CMD`, `CMD-SHELL` 之一。如果是一个字符串，隐含地表示一个 `CMD-SHELL` 前缀。

```
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]
```

如上述示例，但隐含调用 `/bin/sh`，和以下的形式是等效的。

```
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
test: curl -f https://localhost || exit 1
```

要禁用任何在映像内指定的缺省的健康检查向，可以使用 `disable: true`。这和指定 `test: ["NONE"]` 是等效的。

```
healthcheck:
  disable: true
```

### `init`

> since v3.7

在容器中运行一个 init 进程并转发信号。设置为 `true` 为服务使能这个特性。

```
version: "3.8"
services:
  web:
    image: alpine:latest
    init: true
```

> 缺省的 init 进程使用二进制执行文件 [Tini](https://github.com/krallin/tini)，在需要时它将被安装于 daemon主机的位置 `/usr/libexec/docker-init` 。你也可以配置 daemon 使用一个不同的二进制文件，通过 `init-path`，参阅 [configuration option](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)。

### `isolation`

指定一个容器的隔离层级/技术。在 Linux 中，仅支持 `default` 值。在 Windows 中，可以接受的值有：`default`, `process` 和 `hyperv`。

### `labels`

为容器添加元数据标签，参考 [Docker labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/)。可以为其指定一个数组或一个字典。

我们建议你采用反向DNS标注方式来定义你的标签，这可以有效地避免标签名称的冲突。

```
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

### `links`

> 已经是一个遗留特征了。在不久的未来将被移除。
>
> 按：所以我也不精确翻译了:) 太啰嗦了。

链接另一个服务到本容器。可以同时制定服务名称和链接别名（`SERVICE:ALIAS`），也可以略过链接别名。

```
web:
  links:
   - db
   - db:database
   - redis
```

已经被链入的服务将会是主机名（即链接别名 `ALIAS`）可访问的。

对于服务间通讯来说，链接并不是必须的。默认情况下，任何服务都可以通过服务名访问到其他服务。参阅 [Links topic in Networking in Compose](https://docs.docker.com/compose/networking/#links)。

链接也表示一个依赖关系，但这已经是 `depends_on` 的任务了，所以链接也并不必要了。

### `logging`

为服务指定日志转发配置。

```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

`driver` 指定了驱动名称，这和 `--log-driver` 是等效的。

缺省值为 `json-file`。

```
driver: "json-file"
driver: "syslog"
driver: "none"
```

> Note
>
> 使用 `docker-compose up` 和 `docker-compose logs` 检索日志时，只有 `json-file` 和 `journald` 驱动格式的日志才能被打印到控制台，其他日志驱动程序会将日志转发到对应的目的地，本地将不会检索到任何日志输出信息。

可用的转发驱动器可以参考 [https://docs.docker.com/confi...](https://docs.docker.com/config/containers/logging/configure/)。

使用 `option` 指定驱动器的选项，如同 `--log-opt` 那样。例如为 `syslog` 这样指定：

```
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```

缺省的日志转发驱动为 `json-file`。对此可以指定日志切割尺寸以及最多保持的日志历史文件个数：

```
options:
  max-size: "200k"
  max-file: "10"
```

上面显示的这个例子中，日志的文件存储将被限制为 200kB，并且在超出时会被截断到旧历史日志，这样的历史文件将被限制为不超过 10 个，更旧的历史文件将被抛弃。

这里有一个完整的 `docker-compose.yml` 文件实例，演示了如何限制日志存储空间：

```
version: "3.8"
services:
  some-service:
    image: some-service
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

> 有效可用的日志选项有赖于具体使用的日志驱动程序。
>
> 上面的控制日志文件个数及其大小的日志选项，适用于 [json-file driver](https://docs.docker.com/config/containers/logging/json-file/)。它们可能并不适用于别的日志驱动程序。要了解完整的针对每个日志驱动程序可用的日志选项，请查阅 [logging drivers](https://docs.docker.com/config/containers/logging/configure/) 文档。

### `network_mode`

网络模型。

和 `--network` 的取值相同。但额外支持 `service:[service name]` 模式。

```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。
>
> **NOTE**: `network_mode: "host"` 不能和 [links](https://docs.docker.com/compose/compose-file/#links) 混用。

### `networks`

要加入的网络。目标网络是在 `docker-compose.yml` 顶级的 `networks` 项中定义的。

```
services:
  some-service:
    networks:
     - some-network
     - other-network
```

#### ALIASES

指定网络中该服务的别名（也即主机名）。相同网络中别的容器可以使用服务名或者服务别名来连接到该服务的容器实例。

既然 `aliases` 是网络范围（网络域）内的，同一个服务在不同网络中可以有不同的别名。

> Note
>
> 一个网络域别名，可以被多个容器、甚至是多个服务所共享——你可以在不同容器、不同服务之间使用重名的别名。如果你这么做了，别名会被解决为哪个确定的容器是无法保证的。

别名定义的格式如同这样：

```
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

一个更复杂而完整的例子：下面提供了三个服务：``web`, `worker`, 和 `db`，分别属于两个网络：`new` 和 `legacy`。通过 `new` 网络中的主机名 `db` 或 `database` 来访问，或者 `legacy` 网络的主机名 `db` 或 `mysql`，都可以访问到 `db` 服务。

```
version: "3.8"

services:
  web:
    image: "nginx:alpine"
    networks:
      - new

  worker:
    image: "my-worker-image:latest"
    networks:
      - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

#### IPV4_ADDRESS, IPV6_ADDRESS

指定一个静态IP地址。

注意相应的顶级网络配置中，必须有 `ipam` 块对子网进行了配置且静态IP地址符合该子网定义。

> If IPv6 addressing is desired, the [`enable_ipv6`](https://docs.docker.com/compose/compose-file/compose-file-v2/##enable_ipv6) option must be set, and you must use a [version 2.x Compose file](https://docs.docker.com/compose/compose-file/compose-file-v2/#ipv4_address-ipv6_address). *IPv6 options do not currently work in swarm mode*.

一个例子是：

```
version: "3.8"

services:
  app:
    image: nginx:alpine
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"
        - subnet: "2001:3984:3989::/64"
```

### `pid`

```
pid: "host"
```

设置服务使用主机的 PID 模式。这使得容器内的服务进程和宿主机操作系统级能够共享 PID 地址空间。这是一个典型的 Linux/Unix 操作系统概念，因此这里不再展开叙述了。这样的共享的作用，可以使能安全的、借助PID地址空间的 IPC 通讯。

### ports

暴露端口到宿主机。

> **Note**: 端口暴露功能和 `network_mode: host` 不能兼容。

#### 短格式

可以同时指定宿主机和容器端口 (`HOST:CONTAINER`) 以完成映射，也可以仅指定容器端口以自动映射为~~相同的主机端口~~一个临时端口（从32768开始）。

> Note
>
> 当采用 `HOST:CONTAINER` 格式来映射端口时，如果使用的容器端口小于60的话，你可能会**接受**到一个错误。这是因为 YAML 解析器将格式 `xx:yy` 视为一个 60 进制的数值。这挺荒谬的，是不是？由于这个原因，我们不得不推荐你在端口号这里总是使用引号做包围，令其成为一个 string 值，以免收到不如预期的反应。

```
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

#### 长格式

允许进行冗长的定义：

- `target`: 指定容器内的端口号
- `published`: 指定暴露给 docker 宿主机的端口号
- `protocol`: 协议 (`tcp` or `udp`)
- `mode`: `host` 表示每个节点的端口号都发布为宿主机端口，`ingress` 专用于 swarm 集群，所有节点的端口会被负载均衡为宿主机端口。

```
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

> 意义明显，所以略过解说。
>
> **NOTE**：长格式仅在 v3.2 及之后有效。

### `restart`

`no` 是默认的重启策略。此时无论容器怎么退出、怎么失败也不会被自动重启。

指定 `always` 时任何情况下容器都会被重启。

`on-failure` 策略可在容器失败退出时才重启。

```
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。（此时可以使用
>
>
>
> ```
> restart_policy
> ```
>
>
>
> 达到目的）
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### `secrets`

从每个服务配置中，授权访问顶级 `secrets` 定义的表项。支持长短两个格式。

#### 短格式

短格式仅指定敏感内容的名字。这使得容器能够挂载对应内容到 `/run/secrets/<secret_name>` 位置并访问它。

下面的例子使用短格式，让 `redis` 能够访问 `my_secret` 和 `my_other_secret`。`my_secret` 的具体内容被定义在 `./my_secret.txt`，`my_other_secret` 被定义为外部资源，例如通过 `docker secret create` 方式预先定义。如果找不到对应的外部资源，stack部署将会失败并抛出一个 `secret not found` 错误。

```
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

#### 长格式

长格式可以更精细地定义敏感内容如何被用于 stack 内容器。

- `source`： 敏感内容在 Docker 中所被定义的名字。
- `target`: 被挂载到容器内 `/run/secrets/` 中的文件名。如果没指定则使用 `source` 名。
- `uid` & `gid`: 容器内挂载的文件的 UID 和 GID。如未指定则为0。在 Windows 中无效。
- `mode`: 容器内挂载的文件的八进制许可权限。在 Docker 1.13.1 中默认值为 `0000`，但在更新的版本中为 `0444`。挂载的文件是不可写的。执行位可以被设置，但一般情况下没有意义。

下面是一个例子：

```
version: "3.8"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

长短格式时可以被混用的，如果你在定义多个敏感内容。

### security_opt

为每个容器覆盖掉默认的标签语义。

```yaml
security_opt:
  - label:user:USER
  - label:role:ROLE
```

```yaml
security_opt:
  - seccomp:unconfined
  - apparmor:unconfined
```

通常这和 seccomp 有关，这会是与安全配置有关的一个冗长的话题，故而此处不做展开。

### stop_grace_period

指定一个等待时长，如果容器未能拦住 `SIGTERM` 信号（或者通过 `stop_signal` 定义的别的信号）正常地关闭自己，则在此时长之后强制清除容器实例的相应进程（通过 `SIGKILL` 信号）。

```
stop_grace_period: 1s
stop_grace_period: 1m30s
```

默认时，将会等候 10s 。

### `stop_signal`

设置一个替代信号以正常关闭容器实例。默认时使用 `SIGTERM` 信号.

```
stop_signal: SIGUSR1
```

### `sysctls`

为容器设定内核参数。可以使用一个数组或字典。

```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### `tmpfs`

> v2 的解说在 v3.8 的原文里已经被删除了：
>
> > since v2
>
> 挂载一个临时文件系统到容器中。可以是一个单一值或一个列表。
>
> ```
> tmpfs: /run
> tmpfs:
>   - /run
>   - /tmp
> ```
>
> > NOTE
> >
> > : 这些选项在部署一个栈到 swarm mode 时被忽略。
> >
> > 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

> since v3.6

挂载一个临时文件系统到容器中。可以使用一个单一值或一个数组。

```
tmpfs: /run

tmpfs:
  - /run
  - /tmp
```

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

挂载一个临时文件系统到容器中。Size参数可以指定文件系统尺寸的字节大小。默认值为无限。

```
 - type: tmpfs
     target: /app
     tmpfs:
       size: 1000
```

### `ulimits`

覆盖容器内指定的默认的 ulimits 值。可以指定一个整数作为单一的 limit 限制，或者指定一个 mapping 以分别表示 soft/hard limit 限制。

```
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

### `userns_mode`

```yaml
userns_mode: "host"
```

禁用用户名字空间。如果 Docker daemon 被配置运行在一个 user namespace 之中的话。

> NOTE
>
> : 这些选项在部署一个栈到 swarm mode 时被忽略。
>
> 也参阅 [deploying a stack in swarm mode](https://docs.docker.com/engine/reference/commandline/stack_deploy/) 。

### volumes

挂载宿主机路径或者命名卷。

可以挂载一个主机路径到一个服务中，此时无需在顶级 `volumes` 中对其进行定义。

如果想要重用一个卷到多个服务，那么应该在顶级 `volumes` 中定义它并命名它。

可以在 [services, swarms, and stack files](https://docs.docker.com/compose/compose-file/#volumes-for-services-swarms-and-stack-files) 中使用命名卷。

下面的例子示意了一个命名卷 `my_data` ，且被用于 `web` 服务。在 `web` 中也使用一个主机文件夹 `./static` 到容器内的挂载；在 `db` 中挂载了一个主机文件到容器内的对应文件，并使用了另一个命名卷 `dbdata`。

```yaml
version: "3.8"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

#### 短格式

可以使用 `HOST:CONTAINER` 格式，或者附带一个访问模式 `HOST:CONTAINER:ro`。

短格式的语法为 `[SOURCE:]TARGET[:MODE]`。`SOURCE` 可以是一个主机路径，也可以是一个卷名。`TARGET` 是一个容器内路径 T，主机路径将被挂载到该路径 T。`MODE` 可以是 `ro` 或者 `rw`，分别代表着 只读 和 可读写。

可以挂载一个主机中的相对路径，此路径是相对于 docker compose 文件而被展开的。相对路径应该总是以 `.` 或者 `..` 开始。

```
volumes:
  # Just specify a path and let the Engine create a volume
  - /var/lib/mysql

  # Specify an absolute path mapping
  - /opt/data:/var/lib/mysql

  # Path on the host, relative to the Compose file
  - ./cache:/tmp/cache

  # User-relative path
  - ~/configs:/etc/configs/:ro

  # Named volume
  - datavolume:/var/lib/mysql
```

#### 长格式

长格式可以进行更精细的控制。

- `type`: 挂载类型 为 `volume`, `bind`, `tmpfs` 和 `npipe`
- `source`：挂载的源位置。可以是一个主机路径，一个定义于顶级 volumes 中的卷名称，等等。如果是挂载 `tmpfs` 则此参数无意义。
- `target`: 容器内的挂载点路径。
- `read_only`: 布尔值以设定卷的可写性。
- `bind`: 配置附加的 bind 选项。
  - `propagation`: 用于 bind 的传播模式。
- `volume`: 配置附加的 卷 选项
  - `nocopy`：布尔量以禁用数据复制（默认时当卷被首次创建时，容器内的内容将被复制到卷内）
- `tmpfs`: 配置附加的 tmpfs 选项
  - `size`: tmpfs的容量，按字节数。
- `consistency`：挂载的一致性要求：`consistent` 主机和容器有同样的视图，`cached` 读操作被缓冲，主机视图为主体，`delegated` 读写操作被缓冲，容器视图为主体。

```
version: "3.8"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```

> 长格式在 v3.2 之后可用
>
> **Note**
>
> 在做绑定主机目录并挂载到容器操作时，长格式语法需要你提前将主机文件夹准备就绪。使用短格式的话，对应文件夹将被就地创建，如果它尚未存在的话。详情请参阅 [bind mounts documentation](https://docs.docker.com/storage/bind-mounts/#differences-between--v-and---mount-behavior)。

#### VOLUMES FOR SERVICES, SWARMS, AND STACK FILES

当工作在 services, swarms, 或 `docker-stack.yml` 场景下，要注意一个服务在 swarm 中可能被部署到任意节点，而每当服务被更新之后再次启动时，可能已经不再在原来的节点上了。

当指定名称的卷并不存在时，Docker会自动创建一个匿名卷用于引用的服务。匿名卷是不可持久化的，因此当关联的容器实例退出并被移除时，匿名卷也会被销毁。

如果想要持久化你的数据，采用命名卷以及选择恰当的卷驱动程序，这个驱动应该是跨主机的，这样数据才能在不同的主机之间漫游。否则的话，你应该设置服务的约束条件以便该服务只会被部署到特定的节点上，这些节点上有相应的卷服务在正确工作。

作为一个例子，[votingapp sample in Docker Labs](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp.md) 的 `docker-stack.yml` 文件定义了 `db` 服务，运行着 postgresql。它使用了一个命名卷 `db-data` 来持久化数据库数据，这个卷被通过swarm约束在只能运行在 `manager` 这个节点上，因此一切疑难都不存在了。下面是源码：

```
version: "3.8"
services:
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
```

#### CACHING OPTIONS FOR VOLUME MOUNTS (DOCKER DESKTOP FOR MAC)

在 Docker 17.04 CE Edge 以及其后版本中（乃至于 17.06CE Edge 和 Stable 版本），你可以配置容器和主机之间卷如何被同步的一致性约束参数。这些标志包括：

- `consistent` 完全一致。主机和容器有同样的视图，这是默认的策略。
- `cached` 宿主机为准。对卷的读操作被缓冲，主机视图为主体，
- `delegated` 容器为准。对卷的读写操作被缓冲，容器视图为主体。

这是专为 Docker Desktop for Mac 而适配的。由于已经知道的 `osxfx` 的关于文件共享特性原因，合理的设置一致性标志能够改善容器内外访问挂载卷时的性能问题。

下面是一个`cached`卷的例子：

```
version: "3.7"
services:
  php:
    image: php:7.1-fpm
    ports:
      - "9000"
    volumes:
      - .:/var/www/project:cached
```

对于读写操作均被缓冲的情况，即使容器中发生了什么修改（对于向PHP Website这样的典型架构来说，./config.php 经常是可能被写入的），也不会立即体现到宿主机中来，容器中的写入将会被堆积。

卷在容器内外的一致性问题，应该参考 [Performance tuning for volume mounts (shared filesystems)](https://docs.docker.com/docker-for-mac/osxfs-caching/)。

> 在这里我未能原样翻译，因为那样会带来较长的篇幅，我尚未能就此问题组织好语言。

### `domainname`, `hostname`, `ipc`, `mac_address`, `privileged`, `read_only`, `shm_size`, `stdin_open`, `tty`, `user`, `working_dir`

这些配置具有单一值。和 `docker run` 的相应命令行参数相对应。`mac_address` 已经是被遗弃的设定。

```
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```

## 指定时间段 duration

有的配置选项，例如 `interval` 或者 `timeout`（都是 `check `的子选项），接受一个字符串风格的时间周期或时间段的参数值。它们应该具有这样的格式：

```
2.5s
10s
1m30s
2h32m
5h34m56s
```

可以为数值附加的后缀单位有 `us`, `ms`, `s`, `m`, 以及 `h`。

含义自明。

## 指定字节值

有的配置选项，例如 `build` 的子选项 `shm_size`，接受一个字符串分隔的容量尺寸参数值。它们应该具有这样的格式：

```
2b
1024kb
2048k
300m
1gb
```

有效的后缀单位包括 `b`, `k`, `m` 和 `g`。此外，`kb`, `mb` 和 `gb` 也是合法的。纯粹的十进制数值并不合法。

## 卷编排格式手册 - `volumes`

顶级的 volumes 章节可以声明和创建命名卷（无需使用 `volume_from`），这些卷能够被用于在 service 章节下的 [volumes](https://segmentfault.com/a/1190000023284156#volumes) 小节中被引用。所以我们可以重用它们，甚至能够跨越多个 services。docker命令的 [docker volume](https://docs.docker.com/engine/reference/commandline/volume_create/) 子命令有更多的参考信息。

关于卷的使用，也可以参考 [Use volumes](https://docs.docker.com/engine/admin/volumes/volumes/) 和 [Volume Plugins](https://docs.docker.com/engine/extend/plugins_volume/)。

这里有一个示例，包含了两个服务，数据库的数据存储文件夹在两个服务之间被共享，因而数据库可以使用这个存储文件夹，而备份服务同样可以操作它以完成备份任务：

```
version: "3.8"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```

顶级 `volumes` 章节下的条目可以是空，无需指定细节，这样的话，默认的卷驱动程序将被应用（通常都会是 `local` 卷驱动）。

但你也可以通过下面的参数对其进行定制：

### `driver`

指定哪一个卷驱动程序会被采用。一般来说，默认值会是 `local`。如果卷驱动程序无效、或不能工作，在 `docker-compose up` 时 Docker Engine将会返回一个错误。

```
driver: foobar
```

### `driver_opts`

可选地指定一组键值对参数集，这些参数将被传递给卷驱动程序。所以这些参数集是和卷驱动程序相关的，请参考卷驱动程序的有关文档。

```
volumes:
  example:
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```

### `external`

如果设置为 `true`，表示相应的卷是在 compose 编排文件之外被创建就绪的。此时 `docker-compse up` 将不会尝试创建这个卷，而如果该卷尚未存在则会返回一个错误。

对于 v3.3 以及更低的 compose 编排格式版本而言，`external` 不可以被用于与其他卷配置参数组合使用，例如 `driver`, `driver_opts`, `labels` 等等。但对于 v3.4 以及更高版本来说不再有此限制。

下面的示例中，Compose 查找一个名为 `data` 的外部卷并挂载它到 `db` 服务中，而不是尝试创建一个名为 `[projectname]_data` 的新卷。

```
version: "3.8"

services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
    external: true
```

> `external.name` 在 v3.4+ 已被废弃，这之后直接使用 `name`。

你也可以单独指定卷名字（这时，`data` 被认为是该卷在当前编排文件中被引用时的 卷别名）：

```
volumes:
  data:
    external:
      name: actual-name-of-volume
```

> External volumes are always created with docker stack deploy
>
> 在使用 [docker stack deploy](https://docs.docker.com/compose/compose-file/#deploy) 部署到 swarm 中时，外部卷如果不存在，则总是自动被创建。进一步的有关信息请参阅 [moby/moby#29976](https://github.com/moby/moby/issues/29976)、

### `labels`

使用 [Docker labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/) 为容器添加元数据。可以是数组格式或者字典格式。

我们建议你使用反向DNS标注方式，为你的元数据表键添加反向域名前缀，从而避免潜在的和其它应用的相同名字的表键发生冲突：

```
labels:
  com.example.description: "Database volume"
  com.example.department: "IT/Ops"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Database volume"
  - "com.example.department=IT/Ops"
  - "com.example.label-with-empty-value"
```

### `name`

> since v3.4+

为卷指定一个自定义的名字。名字的值可被用于解决具有特殊字符名字的卷。注意该值被原样使用，引号不会被忽略，也不会被添加上栈名字前缀。

```
version: "3.8"
volumes:
  data:
    name: my-app-data
```

`name` 可以被与 `external` 相组合：

```
version: "3.8"
volumes:
  data:
    external: true
    name: my-app-data
```

## 网络编排格式手册 - `networks`

顶级章节 `networks` 使得你可以配置想要创建和使用的网络（Compose内网）。

- 完整的在 Compose 中使用 Docker 网络环境特性的有关说明，以及所有的网络驱动程序选项，请参考 [Networking guide](https://docs.docker.com/compose/networking/)。
- 对于 [Docker Labs](https://github.com/docker/labs/blob/master/README.md) 的和网络相关的辅导用例，请仔细阅读 [Designing Scalable, Portable Docker Container Networks](https://github.com/docker/labs/blob/master/networking/README.md)。

### `driver`

指定该网络的驱动程序。

缺省的驱动程序由 Docker Engine 的启动参数所指定。通常情况下，启动参数内置为在单节点宿主机上使用 `bridge` 驱动，而在 `swarm mode` 中使用 `overlay` 驱动。

如果驱动程序不可用，Docker Engine 将会返回一个错误。

```
driver: overlay
```

#### `bridge`

缺省时 Docker 在每个宿主机节点上使用 `bridge` 驱动。有关桥接网络是如何工作的，可以参考 [Docker Labs](https://github.com/docker/labs/blob/master/README.md) 的和网络相关的辅导用例：[Bridge networking](https://github.com/docker/labs/blob/master/networking/A2-bridge-networking.md)。

#### `overlay`

`overlay` 驱动在多个 `swarm mode` 节点之间建立一个命名子网，这是一个跨主机的虚拟网络。

- 在 `swarm mode` 中如何建立 `overlay` 网络并籍此令服务跨主机正确工作，请参考 [Docker Labs](https://github.com/docker/labs/blob/master/README.md) 的和网络相关的辅导用例： [Overlay networking and service discovery](https://github.com/docker/labs/blob/master/networking/A3-overlay-networking.md)。
- 如果想要深究 `overlay` 是如何跨主机完成虚拟网络构建和报文如何流转的，可以参考 [Overlay Driver Network Architecture](https://github.com/docker/labs/blob/master/networking/concepts/06-overlay-networks.md)。

#### `host` 或 `none`

使用主机网络栈，或者不使用网络。

和命令行参数 `--net=host` 以及 `--net=none` 是等价的。

这两种驱动及网络模型只能被用于 `docker stack` 之中。如果你正在使用 `docker compose` 相关指令，请使用 `network_mode` 来指定它们。

If you want to use a particular network on a common build, use [network] as mentioned in the second yaml file example.

使用内建的网络模型，例如 `host` 和 `none`，语法上有一点点需要注意的地方：如果用 `host` 或 `none` 这样的名字定义一个外部网络（注意你并不需要真的创建他们，这两者都属于Docker内置的网络模型），那么在 Compose 编排文件中引用它们时你需要使用 `hostnet` 或 `nonet`，如同这样：

```
version: "3.8"
services:
  web:
    networks:
      hostnet: {}

networks:
  hostnet:
    external: true
    name: host

---
services:
  web:
    ...
    build:
      ...
      network: host
      context: .
      ...
services:
  web:
    ...
    networks:
      nonet: {}

networks:
  nonet:
    external: true
    name: none
```

### `driver_opts`

指定一组键值对表示的选项集，以传递给网络驱动程序。它们是和驱动程序密切相关的，所以具体的可用参数应该参考对应的驱动程序文档。

```
driver_opts:
  foo: "bar"
  baz: 1
```

### `attachable`

> since v3.2+

只能用于 `driver: overlay` 的场景。

如果被设置为 `true`，独立运行的容器也能被附着在该网络之中。如果独立运行的容器实例被附着到了一个 overlay 网络中，那么容器中的服务与单独的容器实例之间能够互相通讯。请注意你甚至可以附着其他 Docker daemon 中的容器实例到本 overlay 网络中。

```
networks:
  mynet1:
    driver: overlay
    attachable: true
```

### `enable_ipv6`

在该网络/子网中启用 IPv6。

> 在 v3+ 中不被支持。
>
> `enable_ipv6` 需要你使用 v2 的编排格式，而且也不能被用于 swarm mode 中。

### `ipam`

自定义 IPAM 配置。每一项子配置都是可选参数。

- `driver`: 自定义 IPAM 驱动程序，而不使用默认值
- `config`: 一个列表，包含一到多个配置块。每个配置块具有下列子参数：
  - `subnet`: CIDR格式的子网定义，以划定一个网段。

一个完整的例子：

```
ipam:
  driver: default
  config:
    - subnet: 172.28.0.0/16
```

> **NOTE**：附加IPAM如 `gateway` 只在 v2 中可用。

### `internal`

默认时，Docker也会连接到一个桥接网络以提供外部可连接性。如果你想建立一个外部的隔离的 overlay 网络，请设置本选项为 `true`。

### `labels`

使用 [Docker labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/) 为容器添加元数据。可以是数组格式或者字典格式。

我们建议你使用反向DNS标注方式，为你的元数据表键添加反向域名前缀，从而避免潜在的和其它应用的相同名字的表键发生冲突：

```
labels:
  com.example.description: "Financial transaction network"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Financial transaction network"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

### `external`

如果设置为 `true`，那么本网络是在 Compose 编排文件之外被创建和管理的。此时 `dockercompose up` 不会试图创建它，如果网络并不存在则返回一个错误。

对于 v3.3 以及更低版本的格式，`external` 不可与 `driver`, `driver_opts`, `ipam`, `internal` 等连用。此限制在 v3.4+ 之后被取消。

下面的例子里，`proxy` 是一个外部世界中的网关，Compose将会寻找通过 `docker network create outside` 所建立的 `outside` 外部网络，而不是试图去自动建立一个名为 `[projectname]_outside` 的新网络：

```
version: "3.8"

services:
  proxy:
    build: ./proxy
    networks:
      - outside
      - default
  app:
    build: ./app
    networks:
      - default

networks:
  outside:
    external: true
```

> `external.name` 在 v3.5 及之后已经被废弃，请改用 `name`。

你也可以单独指定一个网络名用于在Compose编排文件内被引用。

### `name`

> since v3.5

为网络设置一个自定义名字。名字的值可被用于解决具有特殊字符名字的卷。注意该值被原样使用，引号不会被忽略，也不会被添加上栈名字前缀。

```
version: "3.8"
networks:
  network1:
    name: my-app-net
```

`name` 可以与 `external` 一起连用：

```
version: "3.8"
networks:
  network1:
    external: true
    name: my-app-net
```

## 配置项编排格式手册 - `configs`

顶级的 `configs` 章节声明，定义了一个配置项或者其参考，该配置项可以被授权给栈内服务使用。配置项的来源可以是 `file` 或 `external`。

- `file`: 配置项的内容在一个宿主机文件中。
- `external`: 如果设置为 `true`，表示该配置项已经创建就绪了。Docker将不会试图建立它，而是在起不存在时生成一个 `config not found` 错误。
- `name`: 该配置项在 Docker 中的名字。名字的值可被用于解决具有特殊字符名字的卷。注意该值被原样使用，引号不会被忽略，也不会被添加上栈名字前缀。
- `driver` and `driver_opts`: The name of a custom secret driver, and driver-specific options passed as key/value pairs. Introduced in version 3.8 file format, and only supported when using `docker stack`.
- `template_driver`: The name of the templating driver to use, which controls whether and how to evaluate the secret payload as a template. If no driver is set, no templating is used. The only driver currently supported is `golang`, which uses a `golang`. Introduced in version 3.8 file format, and only supported when using `docker stack`. Refer to [use a templated config](https://docs.docker.com/engine/swarm/configs/#example-use-a-templated-config) for a examples of templated configs.

下面的例子中，当作为栈的一部分被部署时，`my_first_config` 会被自动创建并命名为 `<stack_name>_my_first_config`，至于 `my_second_config` 是已经存在的。

```
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external: true
```

另一种变化是外部配置项带有 `name` 定义的情况，此时该配置项在 Compose 中可以被以 `redis_config` 为名进行参考和使用：

```
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external:
      name: redis_config
```

你仍需在栈内为每个服务声明 `configs` 章节以获得访问配置项的权利，参考 [grant access to the config](https://segmentfault.com/a/1190000023284156#configs)。

## 敏感信息项编排格式手册 - `secrets`

顶级的 `secrets` 章节声明，定义了一个敏感信息项或者其参考，该敏感信息项可以被授权给栈内服务使用。敏感信息项的来源可以是 `file` 或 `external`。

- `file`: 敏感信息项的内容在一个宿主机文件中。
- `external`: 如果设置为 `true`，表示该敏感信息项已经创建就绪了。Docker将不会试图建立它，而是在起不存在时生成一个 `secret not found` 错误。
- `name`: 该敏感信息项在 Docker 中的名字。名字的值可被用于解决具有特殊字符名字的卷。注意该值被原样使用，引号不会被忽略，也不会被添加上栈名字前缀。
- `template_driver`: The name of the templating driver to use, which controls whether and how to evaluate the secret payload as a template. If no driver is set, no templating is used. The only driver currently supported is `golang`, which uses a `golang`. Introduced in version 3.8 file format, and only supported when using `docker stack`.

下面的例子中，当作为栈的一部分被部署时，`my_first_secret` 会被自动创建并命名为 `<stack_name>_my_first_secret`，至于 `my_second_secret` 是已经存在的。

```
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
```

另一种变化是外部配置项带有 `name` 定义的情况，此时该配置项在 Compose 中可以被以 `redis_secret` 为名进行参考和使用。

#### Compose File v3.5 及更高版本

```
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
    name: redis_secret
```

### Compose File v3.4 和更低版本

```
my_second_secret:
    external:
      name: redis_secret
```

你仍需在栈内为每个服务声明 `secret` 章节以获得访问敏感信息项的权利，参考 [grant access to the secret](https://segmentfault.com/a/1190000023284156#secrets)。

## 变量替换

Compose编排文件中可以使用环境变量。当 `docker-compose` 运行时，Compose 从 Shell 环境变量中抽取变量值。例如，假设操作系统的环境变量中包含 `POSTGRES_VERSION=9.3` 的定义，那么以下定义

```
db:
  image: "postgres:${POSTGRES_VERSION}"
```

等价于

```
db:
  image: "postgres:9.3"
```

如果环境变量并不存在或者为空串，那么它就被当做是空字符串。

你可以通过 `.env` 文件来为环境变量设定缺省值。Compose 将会自动查找当前文件夹中的 `.env` 文件以获得环境变量的值。

> **IMPORTANT**: 注意 `.env` 文件只在 `docker-compose up` 场景中有效，而在 `docker stack deploy` 时它并不会被使用。

两种语法 `$VARIABLE` 和 `${VARIABLE}` 都是可用的。此外在 v2.1 格式中，类似于 Shell 语法的如下形式也可以被使用：

- `${VARIABLE:-default}` 将会返回 `default`，如果环境变量 `VARIABLE` 为空字符串或未被设置的话。
- `${VARIABLE-default}` 将会返回 `default`，如果环境变量 `VARIABLE` 未被设置的话。

类似地，下面的语法有助于指定一个明确的值：

- `${VARIABLE:?err}` 将会产生错误信息 `err`，如果环境变量 `VARIABLE` 为空或未被设置的话。
- `${VARIABLE?err}` 将会产生错误信息 `err`，如果环境变量 `VARIABLE` 未被设置的话。

其他的 Shell 语法特性并未被支持，例如 `${VARIABLE/foo/bar}`。

如果需要一个美元符号的话，请使用 `$$`。此时 `$$` 不再参与环境变量替换的解释。如下例：

```
web:
  build: .
  command: "$$VAR_NOT_INTERPOLATED_BY_COMPOSE"
```

如果你忘记了这个规则而使用了一个 `$` 单个字符的话，Compose 会警告你：

```
The VAR_NOT_INTERPOLATED_BY_COMPOSE is not set. Substituting an empty string.
```

## 扩展字段

> since v3.4

通过扩展字段，能够重用编排配置片段。它们可以是自由的格式，前提是你将它们定义在 yaml 文档的顶级，并且其章节名以 `x-` 开头：

```
version: '3.4'
x-custom:
  items:
    - a
    - b
  options:
    max-size: '12m'
  name: "custom"
```

> **NOTE**
>
> 从 v3.7 开始（对于 3.x 系列），或者从 v2.4 开始（对于 2.x 系列），扩展字段也可以被放在 服务，卷，网络，配置项以及敏感信息项顶级章节之下的第一级。
>
> 如同这样：
>
> ```
> version: '3.7'
> services:
> redis:
>  # ...
> x-custom:
>  items:
>       - a
>       - b
>     options:
>       max-size: '12m'
>     name: "custom"
> ```

所谓的自由格式，是指这些定义并不被 Compose 所解释。然而当你在某个地方插入它们的引用时，它们会被展开到插入点，然后再结合上下文被 Compose 解释具体的语义。这使用了 [YAML anchors](http://www.yaml.org/spec/1.2/spec.html#id2765878) 语法。

例如，如果你的多个服务都会使用相同的日志记录选项：

```
logging:
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file
```

你可以这样定义：

```
x-logging:
  &default-logging
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file

services:
  web:
    image: myapp/web:latest
    logging: *default-logging
  db:
    image: mysql:latest
    logging: *default-logging
```

通过 [YAML merge type](http://yaml.org/type/merge.html) 语法，你也可以在插入扩展字段定义是覆盖某些子选项。例如：

```
version: '3.4'
x-volumes:
  &default-volume
  driver: foobar-storage

services:
  web:
    image: myapp/web:latest
    volumes: ["vol1", "vol2", "vol3"]
volumes:
  vol1: *default-volume
  vol2:
    << : *default-volume
    name: volume02
  vol3:
    << : *default-volume
    driver: default
    name: volume-local
```

## Compose 文档参考
