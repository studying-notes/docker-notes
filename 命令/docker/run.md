---
date: 2020-09-19 21:32:37  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "docker run 命令详解"  # 文章标题
# description: "文章描述"
url:  "posts/docker/cmd/docker/run"  # 设置网页永久链接
tags: [ "docker", "docker-cli"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

> 在新容器中运行命令

- [基本信息](#基本信息)
  - [用法](#用法)
  - [选项](#选项)
  - [扩展信息](#扩展信息)
- [后台执行 vs 前台执行](#后台执行-vs-前台执行)
  - [后台执行](#后台执行)
  - [前台执行](#前台执行)
- [容器标识](#容器标识)
  - [名称](#名称)
  - [引用方式](#引用方式)
  - [输出容器 ID](#输出容器-id)
- [命名空间](#命名空间)
  - [PID 命名空间设置](#pid-命名空间设置)
  - [UTS 命名空间设置](#uts-命名空间设置)
  - [IPC 命名空间设置](#ipc-命名空间设置)
- [网络设置](#网络设置)
  - [桥接网络](#桥接网络)
  - [主机网络](#主机网络)
- [重启策略](#重启策略)
- [退出清理](#退出清理)
- [安全标签](#安全标签)
- [指定初始化进程](#指定初始化进程)
- [运行时资源限制](#运行时资源限制)
  - [Linux 内存限制组合](#linux-内存限制组合)
  - [内核内存限制](#内核内存限制)
  - [限制匿名页面交换](#限制匿名页面交换)
  - [Windows 下的内存限制](#windows-下的内存限制)
  - [CPU 分配限制](#cpu-分配限制)
  - [CPU 周期限制](#cpu-周期限制)
  - [限制 CPUs（多个 CPU）](#限制-cpus多个-cpu)
  - [块设备 IO 带宽限制](#块设备-io-带宽限制)
- [附加组](#附加组)
- [运行时特权](#运行时特权)
- [日志驱动](#日志驱动)
- [覆盖 Dockerfile 镜像默认值](#覆盖-dockerfile-镜像默认值)
- [COMMAND](#command)
- [ENTRYPOINT](#entrypoint)
- [开放端口及网络](#开放端口及网络)
- [设置环境变量](#设置环境变量)
- [容器健康状况检查](#容器健康状况检查)
- [挂载 tmpfs 文件系统](#挂载-tmpfs-文件系统)
- [为容器添加数据卷](#为容器添加数据卷)
- [USER](#user)
- [WORKDIR](#workdir)
- [示例](#示例)
- [设置容器存储驱动参数](#设置容器存储驱动参数)
- [设置容器元数据](#设置容器元数据)
- [访问 NVIDIA GPU](#访问-nvidia-gpu)
- [添加宿主机解析](#添加宿主机解析)
- [设置容器的限额](#设置容器的限额)
- [设置容器退出信号](#设置容器退出信号)
- [超时停止容器](#超时停止容器)
- [指定容器的隔离技术](#指定容器的隔离技术)
- [配置命名空间的内核参数](#配置命名空间的内核参数)
  - [在容器网络命名空间中打开 IP 转发](#在容器网络命名空间中打开-ip-转发)
  - [当前支持的 `sysctls`](#当前支持的-sysctls)

## 基本信息

### 用法

```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

### 选项

参数选项因为太多，另写成了单独的笔记，见[《`docker run` 参数选项》](run-opts.md)。

### 扩展信息

- `docker run` 命令首先在指定镜像之上 `create` 一个可读写的容器层，然后根据指定参数 `start` 这个容器。实际上，`docker run` 等价于 `/containers/create`、`/containers/(id)/start` 两个命令之和
- `docker run` 命令可以与 `docker commit` 结合使用，以更改容器运行的命令。

-----------------------------------------------------------------------------------------

## 后台执行 vs 前台执行

当运行一个容器时，必须指明是后台执行还是默认的前台执行模式。

```
-d=false
```

### 后台执行

为了让容器以后台模式启动，可以指定 `-d=false` 或 `-d` 选项。除非指定了`--rm` 选项，否则按照设计，当运行容器的根进程退出时，后台模式启动的容器才会退出。如果将 `-d` 和 `--rm` 一起使用，则在容器守护进程退出时（以先发生者为准），将删除该容器。

千万将 `service x start` 命令传递给后台运行的容器，比如：

```bash
docker run -d -p 80:80 image service nginx start
```

这成功地在容器内部启动了 `nginx` 服务。然而，当根进程（`service nginx start`）返回结果后，后台模式的容器将按设计停止运行。启动 `nginx` 服务器这样的进程，可以执行以下操作：

```bash
docker run -d -p 80:80 image nginx -g 'daemon off;'
```

对一个后台模式的容器进行 IO 操作，可以使用网络方式或者共享卷，也可以通过 `docker attach` 重新附加命令到该后台容器。

### 前台执行

在前台模式（未指定 `-d` 时的默认设置）下，`docker run` 可以在容器中启动进程并将控制台附加到进程的标准输入、输出和标准错误。 它甚至可以伪装成一个 TTY（这是大多数命令行可执行文件所期望的）然后传递信号。这些都是可配置的：

| 参数 | 描述 |
| ----- | ----- |
| -a=[] | 附加到 `STDIN`，`STDOUT`，`STDERR` |
| -t | 分配一个伪 TTY |
| --sig-proxy=true | 代理所有接收到的信号到进程（仅 non-TTY 模式） |
| -i | 始终保持 `STDIN` 开启 |

若不指定 `-a` 参数，默认附加到 `STDIN` 和 `STDERR`，[源码](https://github.com/moby/moby/blob/4118e0c9eebda2412a09ae66e90c34b85fae3275/runconfig/opts/parse.go#L267)（moby/runconfig/opts/parse.go）如下：

```go
	// If -a is not set, attach to stdout and stderr
	if copts.flAttach.Len() == 0 {
		attachStdout = true
		attachStderr = true
	}
```

或者通过一个或多个 `-a` 指定标准 IO：

```bash
docker run -a stdin -a stdout -i -t ubuntu /bin/bash
echo "test" | docker run -i -a stdin ubuntu cat -
```

对于交互进程，必须同时指定 `-i -t` 来分配一个伪 TTY 给容器进程。当客户端从管道接收标准输入时，禁止指定 `-t` 。比如：

```bash
echo test | docker run -i busybox cat
```

> **提示**：在容器中以 PID 1 运行的进程会被 Linux 特别处理：它会忽略任何带有默认操作的信号。因此，除非进行了相应的编码，否则进程不会在 `SIGINT` 或 `SIGTERM` 上终止。

**将文件管道传送到容器中以进行构建的方式**：构建完成后，将打印容器的 ID，且可以使用 `docker logs` 检索构建日志。

```bash
cat somefile | docker run -i -a stdin mybuilder dobuild
```

## 容器标识

### 名称

有三种方法可以表示同一个容器：

| 标识符类型 | 示例 |
| :------------ | :------------ |
| UUID 长标识符 | “f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778” |
| UUID 短标识符 | “f78375b1c487” |
| 名称 | “irresistibly” |

UUID 标识符由 Docker 守护进程在用户未指定 `--name` 时随机生成。自定义的名称相对容易区分，也可以在 Docker 网络中引用容器。

> **提示**：默认网桥网络上的容器必须通过**名称**彼此连接以进行通信。

### 引用方式

有三种方法可以引用一个容器：

| 引用方式 | 示例 |
| :------------ | :------------ |
| Image | docker run ubuntu |
| Image[:tag] | docker run ubuntu |
| Image[@digest] | docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 |

### 输出容器 ID

为了实现自动化，可以指定 `--cidfile` 参数让 Docker 将容器 ID 写入指定的文件。Docker 会尝试创建指定名称的文件，当文件已存在时会返回错误。

```bash
docker run --cidfile="cidfile" ubuntu
```

当 `docker run` 退出后，文件会被关闭。

## 命名空间

### PID 命名空间设置

`--pid=""` - 设置容器的 PID 命名空间模式。

- `--pid=container:<name|id>` - 加入另一个容器的 PID 命名空间。
- `--pid=host` - 加入宿主机的 PID 命名空间。

默认情况下，所有容器创建新的 PID 命名空间。

**示例**：在容器中运行 htop

1. 制作 Dockerfile

```Dockerfile
FROM alpine:latest
RUN apk add --update htop && rm -rf /var/cache/apk/*
CMD ["htop"]
```

2. 从 Dockerfile 构建镜像

```bash
docker build -t htop .
```

3. 运行 `htop` 镜像

```bash
docker run -it --rm --pid=host htop
```

**示例**：加入另一个容器

```bash
docker run --name redis -d redis
docker run -it --pid=container:redis alpine bash
```

### UTS 命名空间设置

`--uts=""` - 设置容器的 UTS 命名空间模式。

- `--uts=host` - 加入宿主机的 PID 命名空间，指定 `--hostname` 和 `--domainname` 在这种情况下无效。

UTS 命名空间用于设置主机名和对该命名空间中正在运行的进程可见的域。默认情况下，所有容器都创建新的 UTS 命名空间。

### IPC 命名空间设置

`--ipc="MODE"` - 设置容器的 IPC 命名空间模式，支持的模式有：

| 模式 | 描述 |
| :-------------------------- | :-------------------------- |
| ”” | Use daemon’s default. |
| “none” | Own private IPC namespace, with /dev/shm not mounted. |
| “private” | Own private IPC namespace. |
| “shareable” | Own private IPC namespace, with a possibility to share it with other containers. |
| “container: <_name-or-ID_>" | Join another (“shareable”) container’s IPC namespace. |
| “host” | Use the host system’s IPC namespace. |

IPC (POSIX/SysV IPC) 命名空间提供共享内存段、信号量和消息队列的隔离。共享内存段用于加速进程间通信，通过内存而不是管道或网络堆栈。

## 网络设置

- `--dns=[]`：设置容器的自定义 DNS 服务器，默认与宿主机相同
- `--network="bridge"`：将容器加入一个网络
    - `'bridge'`：在 Docker 默认的桥接网络上创建一个网络堆栈
    - `'none'`：禁用外部网络，但 `loopback` 接口仍可用
    - `'container:<name|id>'`：共享另一个容器的网络堆栈
    - `'host'`：共享宿主机的网络堆栈，指定 `--mac-address=""`  在这种模式下无效。
    - `'<network-name>|<network-id>'`：加入用户自定义网络，在这种模式下，参数 `--add-host`、`--hostname`、`--dns`、`--dns-search`、`--dns-option`、`--mac-address`、`--publish`、`--publish-all`、`--expose` 都无效
- `--network-alias=[]`：为容器添加网络范围的别名
- `--add-host="<host:ip>"`：添加新的一行到 `/etc/hosts` (host:IP)，Docker 可能动态更新该文件
- `--mac-address=""`：自定义容器的 MAC 地址，格式为 `12:34:56:78:9a:bc`，手动设置不检查地址是否唯一，默认自动生成
- `--ip=""`：设置容器的 IPv4 地址
- `--ip6=""`：设置容器的 IPv6 地址
- `--link-local-ip=[]`：设置一个或多个容器的本地 IPv4/IPv6 地址

默认情况下，所有容器都启用了网络，它们可以建立任何**传出**连接。可以通过 `docker run --network none` 禁用网络。

**支持的网络类型**：

| 网络 | 描述 |
| :----------------------- | :----------------------- |
| **none** | No networking in the container. |
| **bridge** (default) | Connect the container to the bridge via veth interfaces. |
| **host** | Use the host's network stack inside the container. |
| **container**:<name\|id> | Use the network stack of another container, specified via its *name* or *id*. |
| **NETWORK** | Connects the container to a user created network (using `docker network create` command) |

### 桥接网络

当设置为 `--network="bridge"` 时，容器将加入 Docker 默认的网络，这个网络在宿主机中显示为 `docker0`。Docker 将为该容器创建一对 `veth` 接口，这对接口的一侧将保留在连接到桥接网络的主机上，而另一侧除了 `loopback` 接口之外，还将放置在容器的命名空间内。还将为这个桥接网络上的容器分配一个 IP 地址，流量将通过该网桥路由到容器。

默认情况下，容器可以通过其 IP 地址进行通信。 而通过名称进行通信时必须将它们连接起来。

### 主机网络

当设置为 `--network="host"` 时，容器共享宿主机的全部网络硬件和配置，拥有完全的相关访问权限。容器的主机名将与宿主机系统的主机名相同。但是默认情况下，容器也有其自己的 UTS 名称空间，因此允许指定 `--hostname` 和 `--domainname` 参数，它们只会更改容器内的主机名和域名。同理，`--add-host`，`--dns`，`--dns-search` 和 `--dns-option` 参数也可以在主机网络模式下指定，这些操作仅更新了容器内的 `/etc/hosts` 或 `/etc/resolv.conf` 文件，不会对主机上的文件进行任何更改。

与默认的桥接模式相比，主机模式由于使用主机的本机网络堆栈而提供了显着改善的网络性能，而桥接则必须通过 Docker 守护进程进行一级虚拟化。

## 重启策略

在容器运行时指定 `--restart` 参数，以说明在 Docker 守护进程退出后是否重启该容器的策略。

当容器的重启策略处于活动状态时，它将在 `docker ps` 中显示为 `Up` 或 `Restarting`，或者运行 `docker events` 命令来查看其重启策略是否有效。

Docker 支持以下几种重启策略：

| 策略 | 描述 |
| :---------- | :---------- |
| **no** | 默认参数，退出后不自动重启容器 |
| **on-failure**[:max-retries] | 仅当容器以非零状态码退出时才重新启动，可以指定守护进程尝试重新启动该容器的最大重试次数，不指定则无限次重启 |
| **always** | 始终重新启动容器，守护进程一旦启动，就会无限期地尝试启动该容器，直到启动成功，或者容器被 `docker rm -f` 删除 |
| **unless-stopped** | 始终重新启动容器，守护进程一旦启动，就会无限期地尝试启动该容器，直到启动成功，除非该容器在守护进程停止之前已处于停止状态（`docker stop`） |

```bash
docker run --restart=always redis
```

Docker 在每次重新启动之前都会添加一个不断增加的延迟（从 100 毫秒开始，每次是之前的两倍），以防止服务器溢出。成功启动容器运行至少 10 秒钟后延迟将被重置为默认值 100 毫秒。

已尝试的重启次数可以通过以下方法获知：

```bash
docker inspect -f "{{ .RestartCount }}" container-id
```

容器最近一次被重启（启动）时间可以通过以下方法获知：

```bash
docker inspect -f "{{ .State.StartedAt }}" container-id
```

不可以同时指定 `--restart`、`--rm`。

**退出状态码**

`docker run` 的退出代码提供有关为何容器无法运行或为何退出的信息，退出代码遵循 `chroot` 标准。输出退出状态码示例：

```bash
docker run busybox; echo $?
```

## 退出清理

默认情况下，即使容器退出后，容器的文件系统也会保留。通过设置参数 `--rm` 自动删除容器的文件系统、与该容器关联的匿名卷。。

## 安全标签

- 设置容器的标签用户：`--security-opt="label=user:USER"`
- 设置容器的标签角色：`--security-opt="label=role:ROLE"`
- 设置容器的标签类型：`--security-opt="label=type:TYPE"`
- 设置容器的标签级别：`--security-opt="label=level:LEVEL"`
- 关闭容器的标签限制：`--security-opt="label=disable"`
- 设置要应用于容器的 apparmor 配置文件：`--security-opt="apparmor=PROFILE"`
- 禁用/启用容器进程以获取新特权：`--security-opt="no-new-privileges:true|false"`
- 关闭容器的 seccomp 限制：`--security-opt="seccomp=unconfined"`
- 将白名单的 `syscalls seccomp Json` 文件用作 seccomp 筛选器：`--security-opt="seccomp=profile.json"`

通过在下面的命令中指定级别，可以在容器之间共享相同的配置内容。

```bash
docker run --security-opt label=level:s0:c100,c200 -it fedora bash
```

禁用容器的安全标签：

```bash
docker run --security-opt label=disable -it fedora bash
```

为了对容器内的进程采用更严格的安全策略，可以为容器指定其他自定义的类型，比如执行下面的命令前必须编写定义 `svirt_apache_t` 类型的策略。

```bash
docker run --security-opt label=type:svirt_apache_t -it centos bash
```

阻止容器进程获取其他特权，即 `su`、`sudo` 提权无效，可以执行以下命令：

```bash
docker run --security-opt no-new-privileges -it centos bash
```

## 指定初始化进程

`--init` 标志指示将初始化进程用作容器中的 PID 1 进程。 指定初始化进程可确保在创建的容器内执行初始化系统的通常职责，例如获取僵尸进程。

默认初始化进程是在 Docker 守护进程的系统路径中找到的第一个 `docker-init` 可执行文件。 默认安装中包含的此 `docker-init` 二进制文件由 [tini](https://github.com/krallin/tini) 驱动。

## 运行时资源限制

| 选项 | 描述 |
| :------------------------- | :------------------------- |
| `-m`, `--memory=""` | Memory limit (format: `[]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. Minimum is 4M. |
| `--memory-swap=""` | Total memory limit (memory + swap, format: `[]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. |
| `--memory-reservation=""` | Memory soft limit (format: `[]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. |
| `--kernel-memory=""` | Kernel memory limit (format: `[]`). Number is a positive integer. Unit can be one of `b`, `k`, `m`, or `g`. Minimum is 4M. |
| `-c`, `--cpu-shares=0` | CPU shares (relative weight) |
| `--cpus=0.000` | Number of CPUs. Number is a fractional number. 0.000 means no limit. |
| `--cpu-period=0` | Limit the CPU CFS (Completely Fair Scheduler) period |
| `--cpuset-cpus=""` | CPUs in which to allow execution (0-3, 0,1) |
| `--cpuset-mems=""` | Memory nodes (MEMs) in which to allow execution (0-3, 0,1). Only effective on NUMA systems. |
| `--cpu-quota=0` | Limit the CPU CFS (Completely Fair Scheduler) quota |
| `--cpu-rt-period=0` | Limit the CPU real-time period. In microseconds. Requires parent cgroups be set and cannot be higher than parent. Also check rtprio ulimits. |
| `--cpu-rt-runtime=0` | Limit the CPU real-time runtime. In microseconds. Requires parent cgroups be set and cannot be higher than parent. Also check rtprio ulimits. |
| `--blkio-weight=0` | Block IO weight (relative weight) accepts a weight value between 10 and 1000. |
| `--blkio-weight-device=""` | Block IO weight (relative device weight, format: `DEVICE_NAME:WEIGHT`) |
| `--device-read-bps=""` | Limit read rate from a device (format: `:[]`). Number is a positive integer. Unit can be one of `kb`, `mb`, or `gb`. |
| `--device-write-bps=""` | Limit write rate to a device (format: `:[]`). Number is a positive integer. Unit can be one of `kb`, `mb`, or `gb`. |
| `--device-read-iops=""` | Limit read rate (IO per second) from a device (format: `:`). Number is a positive integer. |
| `--device-write-iops=""` | Limit write rate (IO per second) to a device (format: `:`). Number is a positive integer. |
| `--oom-kill-disable=false` | Whether to disable OOM Killer for the container or not. |
| `--oom-score-adj=0` | Tune container’s OOM preferences (-1000 to 1000) |
| `--memory-swappiness=""` | Tune a container’s memory swappiness behavior. Accepts an integer between 0 and 100. |
| `--shm-size=""` | Size of `/dev/shm`. The format is ``. `number` must be greater than `0`. Unit is optional and can be `b` (bytes), `k` (kilobytes), `m` (megabytes), or `g` (gigabytes). If you omit the unit, the system uses bytes. If you omit the size entirely, the system uses `64m`. |

### Linux 内存限制组合

四种组合方法可以限制用户内存占用：

| 参数 | 描述 |
| :-------- | :-------- |
| **memory=inf, memory-swap=inf** (default) | 无内存和交换空间大小限制（默认，`inf` 可以用 -1 表示） |
| **memory=L<inf, memory-swap=inf** | 指定内存占用上限为 L，但是不限制交换空间的大小（系统支持情况下） |
| **memory=L<inf, memory-swap=2\*L** | 指定内存占用上限为 L，同时限制交换空间上限为 L（内存+交换空间=2*L） |
| **memory=L<inf, memory-swap=S<inf, L<=S** | 指定内存占用上限为 L，同时限制交换空间上限为 S-L（内存+交换空间=S） |

> `memory-swap` 表示物理内存+交换空间的大小，即虚拟内存的大小。

**示例**

1. 限制内存，不限制交换空间

```bash
docker run -it -m 300M --memory-swap -1 ubuntu /bin/bash
```

2. 默认情况下，交换空间与内存大小相同，即虚拟内存是物理内存的两倍

```bash
docker run -it -m 300M ubuntu /bin/bash
```

3. 限制内存，限制交换空间

```bash
docker run -it -m 300M --memory-swap 1G ubuntu /bin/bash
```

内存预留是一种内存软限制，它可以实现更好的内存共享。在正常情况下，容器仅受 `-m` / `--memory` 选项设置的硬限制的约束。设置内存预留后，当 Docker 检测到内存争用或内存不足时，会强制容器将它们的消耗限制在预定范围内，也就是将内存限制在 `--memory-reservation` 的值之下。

建议始终将内存预留值设置为硬限制之下，否则硬限制优先。预留为 0 等于不设置预留。 默认情况下，内存预留与硬限制相同。内存预留是一项软限制功能，不能保证不会超出限制。相反，此功能试图确保在内存竞争激烈时，根据预留提示/设置分配内存。

**示例**

1. 设置硬限制 500M，软限制 200M

```bash
docker run -it -m 500M --memory-reservation 200M ubuntu /bin/bash
```

当容器消耗的内存超过 200M 但是小于 500M 时，下一次系统内存回收机制将尝试将容器内存限制到 200M 以下。

2. 仅设置软限制

```bash
docker run -it --memory-reservation 1G ubuntu /bin/bash
```

尽管容器可以无限制消耗内存，但是每次系统内存回收机制都会尝试将容器内存限制到 1G 以下。

默认情况下，如果发生了 `out-of-memory (OOM)` 错误，内核会杀死进程。为了改变这种行为，可以指定 `--oom-kill-disable` 选项，但只建议在设置了内存限制的情况下禁用 OOM Killer，否则可能导致宿主机内存耗尽，必须重启才能解决。

```bash
docker run -it -m 100M --oom-kill-disable ubuntu /bin/bash
```

参数 `--oom-score-adj` 可以改变内存不足时容器进程被杀死的优先级，负值表示被杀死的可能性小，正数表示被杀死的可能性大。

### 内核内存限制

内核内存与用户内存有根本的不同，因为内核内存不能交换出去。由于无法交换，容器可能会占用太多的内核内存造成系统服务阻塞。内核内存包括:

- stack pages （堆栈页面）
- slab pages
- sockets memory pressure
- tcp memory pressure

内核内存可以在用户内存限制的范围内进行限制。 假设“U”是用户内存限制，“K”是内核限制，有三种可能的方式来设置限制：

| 参数 | 描述 |
| :---------------------------- | :---------------------------- |
| **U != 0, K = inf** (default) | This is the standard memory limitation mechanism already present before using kernel memory. Kernel memory is completely ignored. |
| **U != 0, K < U** | Kernel memory is a subset of the user memory. This setup is useful in deployments where the total amount of memory per-cgroup is overcommitted. Overcommitting kernel memory limits is definitely not recommended, since the box can still run out of non-reclaimable memory. In this case, you can configure K so that the sum of all groups is never greater than the total memory. Then, freely set U at the expense of the system's service quality. |
| **U != 0, K > U** | Since kernel memory charges are also fed to the user counter and reclamation is triggered for the container for both kinds of memory. This configuration gives the admin a unified view of memory. It is also useful for people who just want to track kernel memory usage. |

**示例**

1. 设置总内存大小为 500M，其中内核最多消耗 50M 内存

```bash
docker run -it -m 500M --kernel-memory 50M ubuntu /bin/bash
```

2. 仅限制内核最多消耗 50M 内存

```bash
docker run -it --kernel-memory 50M ubuntu /bin/bash
```

### 限制匿名页面交换

默认情况下，容器的内核可以换出一定比例的匿名页面。可以指定 `--memory-swappiness` 参数设置可交换百分比（0 ~ 100）。0 表示关闭匿名页面交换，100 表示全部匿名页面可交换。未设置则继承父级。

```bash
docker run -it --memory-swappiness=0 ubuntu /bin/bash
```

关闭匿名页面交换可以保留容器的工作区，避免交换性能的损失。

### Windows 下的内存限制

- `process` 隔离技术，Windows 将报告主机系统的全部内存，而不是容器内运行程序的限制

```powershell
PS C:\> docker run -it -m 2GB --isolation=process microsoft/nanoserver powershell Get-ComputerInfo *memory*

CsTotalPhysicalMemory      : 17064509440
CsPhyicallyInstalledMemory : 16777216
OsTotalVisibleMemorySize   : 16664560
OsFreePhysicalMemory       : 14646720
OsTotalVirtualMemorySize   : 19154928
OsFreeVirtualMemory        : 17197440
OsInUseVirtualMemory       : 1957488
OsMaxProcessMemorySize     : 137438953344
```

- `hyperv` 隔离技术，Windows 将创建一个实用虚拟机，该虚拟机的大小足以容纳内存限制，再加上承载容器所需的最小操作系统，该大小为 `Total Physical Memory`。

```powershell
PS C:\> docker run -it -m 2GB --isolation=hyperv microsoft/nanoserver powershell Get-ComputerInfo *memory*

CsTotalPhysicalMemory      : 2683355136
CsPhyicallyInstalledMemory :
OsTotalVisibleMemorySize   : 2620464
OsFreePhysicalMemory       : 2306552
OsTotalVirtualMemorySize   : 2620464
OsFreeVirtualMemory        : 2356692
OsInUseVirtualMemory       : 263772
OsMaxProcessMemorySize     : 137438953344
```

### CPU 分配限制

默认情况下，所有容器拥有相同比例的 CPU 占用时间。可以通过更改容器的 CPU 份额权重参数 `-c` 或 ` --cpu-shares` 来修改此比例，该参数的默认值是 1024。不可以设置为 0，否则重置为 1024。相关性质[之前笔记](../原理/控制组/操作%20CPU%20子系统.md)已有说明。

### CPU 周期限制

默认的 CPU CFS（完全公平调度算法）一个周期为 100ms。可以指定 `--cpu-period` 修改周期长度，一般同时指定 `--cpu-quota` 以限制容器占用。

**示例**：容器在 50ms 的 CPU 运行时间中可获得 25ms

```bash
docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu /bin/bash
```

其实还有更方便的限制方法，就是指定 `--cpus` 参数（默认 0，表示无限制），比如上面的命令等价于：

```bash
docker run -it --cpus=0.5 ubuntu /bin/bash
```

### 限制 CPUs（多个 CPU）

1. 可用 CPU 1、CPU 3

```bash
docker run -it --cpuset-cpus="1,3" ubuntu /bin/bash
```

2. 可用 CPU 0、CPU 1、CPU 2

```bash
docker run -it --cpuset-cpus="0-2" ubuntu /bin/bash
```

3. 在 NUMA 系统上限制可用内存节点

```bash
docker run -it --cpuset-mems="1,3" ubuntu /bin/bash
```

### 块设备 IO 带宽限制

默认情况下，所有容器拥有相同比例的块设备 IO 带宽（blkio），该参数的默认值是 500。为了修改相对宽带份额，可以指定 `--blkio-weight` 参数。该限制仅对直接 IO 有效，不支持带缓冲的 IO。

该参数值有效取值范围为 10 ~ 1000。

**示例**：启动两个容器

```bash
docker run -it --name c1 --blkio-weight 300 ubuntu /bin/bash
docker run -it --name c2 --blkio-weight 600 ubuntu /bin/bash
```

同时执行下面的代码可以看到花费的时间之比与带宽之比是相同的。

```bash
time dd if=/mnt/zerofile of=test.out bs=1M count=1024 oflag=direct
```

参数 `--blkio-weight-device="DEVICE_NAME:WEIGHT"` 可以单独指定某一设备的权重，会覆盖全局设置 `--blkio-weight` 。比如指定块设备默认权重为 `300`，但是单独指定 `/dev/sda` 设备的权重为 `200`：

```bash
docker run -it \
    --blkio-weight 300 \
    --blkio-weight-device "/dev/sda:200" \
    ubuntu
```

参数 `--device-read-bps` 可以限制某个设备的读取速率（比特每秒），比如：

```bash
docker run -it --device-read-bps /dev/sda:1mb ubuntu
```

参数 `--device-write-bps` 可以限制某个设备的写入速率（比特每秒），比如：

```bash
docker run -it --device-write-bps /dev/sda:1mb ubuntu
```

这些参数的格式为 `<device-path>:<limit>[unit]`，速率值必须是一个正整数，可以指定速率单位 `kb` (kilobytes)、`mb` (megabytes) 或 `gb` (gigabytes)。

参数 `--device-write-iops` 可以限制某个设备的写入速率（每秒 IO 次数），比如：

```bash
docker run -ti --device-write-iops /dev/sda:1000 ubuntu
```

## 附加组

`--group-add` - 添加额外的用户组以运行。默认情况下， Docker 容器进程运行时会在补充组中查找指定的用户。如果想要添加更多的用户组，可以指定该参数：

```bash
docker run --rm --group-add audio --group-add nogroup --group-add 777 busybox id
uid=0(root) gid=0(root) groups=10(wheel),29(audio),99(nogroup),777
```

## 运行时特权

- `--cap-add`: 添加 Linux capabilities
- `--cap-drop`: 剥夺 Linux capabilities
- `--privileged=false`: 给予容器扩展的特权
- `--device=[]`: 允许在未指定 `--privileged` 的情况下运行容器内设备

默认情况下，因为禁用了大多数潜在危险的内核功能，Docker 容器内是无特权的，无法运行特权程序，比如无法在容器内运行 Docker 守护进程。因为容器默认不允许访问任何设备，但是一个拥有特权的容器可以访问[全部设备](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt)。

```bash
docker run -t -i --rm ubuntu bash
mount -t tmpfs none /mnt
mount: permission denied

# 禁用了 cap_sys_admin，允许挂载文件系统的权限
```

当执行 `docker run --privileged` 时，Docker 将允许访问主机上的所有设备，并在 AppArmor 或 SELinux 中进行一些配置，以允许容器对主机的访问几乎与在主机上的容器外部运行的进程相同。

```bash
docker run -t -i --privileged ubuntu bash
mount -t tmpfs none /mnt
df -h
Filesystem      Size  Used Avail Use% Mounted on
none            1.9G     0  1.9G   0% /mnt
```

为了只允许访问指定的设备，可以设置 `--device=[]` 参数，但仅在进程隔离的 Windows 容器上受支持，如果容器隔离为 `hyperv` 或在 Windows 上运行 Linux 容器（LCOW）时，该参数无效。比如：

```bash
docker run --device=/dev/snd:/dev/snd ...
```

默认情况下，容器允许 `read`、`write` 和 `mknod` 这些可访问的设备，但也可以通过参数 `:` 指定权限：

```bash
# 只读
docker run --device=/dev/snd:r

# 读写
docker run --device=/dev/snd:rw

# 全部
docker run --device=/dev/snd:rwm
```

参数 `--cap-add` 和 `--cap-drop` 还可以对 capabilities 进行细粒度控制。默认情况下，Docker 有一个保留的默认功能列表。下表列出了 Linux capability 选项，这些选项是默认允许的，但可以删除。

| Capability Key | Capability Description |
| :--------------- | :--------------- |
| SETPCAP | Modify process capabilities. |
| MKNOD | Create special files using mknod(2). |
| AUDIT_WRITE | Write records to kernel auditing log. |
| CHOWN | Make arbitrary changes to file UIDs and GIDs (see chown(2)). |
| NET_RAW | Use RAW and PACKET sockets. |
| DAC_OVERRIDE | Bypass file read, write, and execute permission checks. |
| FOWNER | Bypass permission checks on operations that normally require the file system UID of the process to match the UID of the file. |
| FSETID | Don’t clear set-user-ID and set-group-ID permission bits when a file is modified. |
| KILL | Bypass permission checks for sending signals. |
| SETGID | Make arbitrary manipulations of process GIDs and supplementary GID list. |
| SETUID | Make arbitrary manipulations of process UIDs. |
| NET_BIND_SERVICE | Bind a socket to internet domain privileged ports (port numbers less than 1024). |
| SYS_CHROOT | Use chroot(2), change root directory. |
| SETFCAP | Set file capabilities. |

下表显示了默认情况下不授予的 capabilities，但可以添加。

| Capability Key | Capability Description |
| :-------------- | :-------------- |
| SYS_MODULE | Load and unload kernel modules. |
| SYS_RAWIO | Perform I/O port operations (iopl(2) and ioperm(2)). |
| SYS_PACCT | Use acct(2), switch process accounting on or off. |
| SYS_ADMIN | Perform a range of system administration operations. |
| SYS_NICE | Raise process nice value (nice(2), setpriority(2)) and change the nice value for arbitrary processes. |
| SYS_RESOURCE | Override resource Limits. |
| SYS_TIME | Set system clock (settimeofday(2), stime(2), adjtimex(2)); set real-time (hardware) clock. |
| SYS_TTY_CONFIG | Use vhangup(2); employ various privileged ioctl(2) operations on virtual terminals. |
| AUDIT_CONTROL | Enable and disable kernel auditing; change auditing filter rules; retrieve auditing status and filtering rules. |
| MAC_ADMIN | Allow MAC configuration or state changes. Implemented for the Smack LSM. |
| MAC_OVERRIDE | Override Mandatory Access Control (MAC). Implemented for the Smack Linux Security Module (LSM). |
| NET_ADMIN | Perform various network-related operations. |
| SYSLOG | Perform privileged syslog(2) operations. |
| DAC_READ_SEARCH | Bypass file read permission checks and directory read and execute permission checks. |
| LINUX_IMMUTABLE | Set the FS_APPEND_FL and FS_IMMUTABLE_FL i-node flags. |
| NET_BROADCAST | Make socket broadcasts, and listen to multicasts. |
| IPC_LOCK | Lock memory (mlock(2), mlockall(2), mmap(2), shmctl(2)). |
| IPC_OWNER | Bypass permission checks for operations on System V IPC objects. |
| SYS_PTRACE | Trace arbitrary processes using ptrace(2). |
| SYS_BOOT | Use reboot(2) and kexec_load(2), reboot and load a new kernel for later execution. |
| LEASE | Establish leases on arbitrary files (see fcntl(2)). |
| WAKE_ALARM | Trigger something that will wake up the system. |
| BLOCK_SUSPEND | Employ features that can block system suspend. |

这两个参数都支持 `ALL`，因此某些情况下更方便控制：

```bash
docker run --cap-add=ALL --cap-drop=MKNOD ...
```

为了与网络堆栈进行交互，应该使用 `--cap-add = NET_ADMIN` 来修改网络接口，而不是使用 `--privileged`。

```bash
docker run -it --rm  ubuntu:14.04 ip link add dummy0 type dummy
RTNETLINK answers: Operation not permitted
docker run -it --rm --cap-add=NET_ADMIN ubuntu:14.04 ip link add dummy0 type dummy
```

挂载一个基于 FUSE 的文件系统，需要结合使用 `--cap-add`和 `--device`：

```bash
docker run --rm -it --cap-add SYS_ADMIN sshfs sshfs sven@10.10.10.20:/home/sven /mnt
fuse: failed to open /dev/fuse: Operation not permitted
docker run --rm -it --device /dev/fuse sshfs sshfs sven@10.10.10.20:/home/sven /mnt
fusermount: mount failed: Operation not permitted
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
# sshfs sven@10.10.10.20:/home/sven /mnt
The authenticity of host '10.10.10.20 (10.10.10.20)' can't be established.
ECDSA key fingerprint is 25:34:85:75:25:b0:17:46:05:19:04:93:b5:dd:5f:c6.
Are you sure you want to continue connecting (yes/no)? yes
sven@10.10.10.20's password:
root@30aa0cfaf1b5:/# ls -la /mnt/src/docker
total 1516
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:08 .
drwxrwxr-x 1 1000 1000   4096 Dec  4 11:46 ..
-rw-rw-r-- 1 1000 1000     16 Oct  8 00:09 .dockerignore
-rwxrwxr-x 1 1000 1000    464 Oct  8 00:09 .drone.yml
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:11 .git
-rw-rw-r-- 1 1000 1000    461 Dec  4 06:08 .gitignore
....
```

## 日志驱动

容器的日志驱动程序可以与 Docker 守护进程不同。可以通过 `--log-driver=VALUE` 配置容器的日志驱动，支持下面的选项：

| 驱动      | 描述                                                  |
| :---------- | :----------------------------------------------------------- |
| `none`      | Disables any logging for the container. `docker logs` won’t be available with this driver. |
| `json-file` | Default logging driver for Docker. Writes JSON messages to file. No logging options are supported for this driver. |
| `syslog`    | Syslog logging driver for Docker. Writes log messages to syslog. |
| `journald`  | Journald logging driver for Docker. Writes log messages to `journald`. |
| `gelf`      | Graylog Extended Log Format (GELF) logging driver for Docker. Writes log messages to a GELF endpoint likeGraylog or Logstash. |
| `fluentd`   | Fluentd logging driver for Docker. Writes log messages to `fluentd` (forward input). |
| `awslogs`   | Amazon CloudWatch Logs logging driver for Docker. Writes log messages to Amazon CloudWatch Logs |
| `splunk`    | Splunk logging driver for Docker. Writes log messages to `splunk` using Event Http Collector. |

`docker logs` 命令仅在 `json-file` 和 `journald` 驱动下有效。

## 覆盖 Dockerfile 镜像默认值

当开发人员从Dockerfile构建映像或提交映像时，开发人员可以设置一些默认参数，这些参数在映像作为容器启动时生效。

四个 Dockerfile 命令在运行时无法被覆盖： `FROM`、`MAINTAINER`、`RUN` 和 `ADD`。而其他命令在执行 `docker run` 时大都有相应的选项参数可以覆盖默认值。

## COMMAND

```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

该命令是可选的，一般镜像创建时都提供了默认命令，只需指定新的命令就可以覆盖默认值，但如果还指定了 `--entrypoint`，则将 `COMMAND` 作为 `ENTRYPOINT` 的参数。

## ENTRYPOINT

`--entrypoint=""` - 在运行时执行的默认命令。`ENTRYPOINT` 与 `COMMAND` 相似，它指定了容器启动时运行的可执行文件，但是它有意被设计的难以被覆盖。`ENTRYPOINT` 为容器提供了其默认性质或行为，因此，当设置 `ENTRYPOINT` 时，可以像运行该二进制文件一样运行该容器，并带有默认选项，并且可以通过 `COMMAND` 传递更多选项。但是，有时可能希望在容器内运行其他内容，因此可以在运行时通过使用字符串指定新的 `ENTRYPOINT` 来覆盖默认的 `ENTRYPOINT`。

**示例**

1. 在已设置为自动运行其他程序的容器中运行 Shell

```bash
docker run -it --entrypoint /bin/bash example/redis
```

2. 将更多参数传递给  `ENTRYPOINT`

```bash
docker run -it --entrypoint /bin/bash example/redis -c ls -l
docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
```

3. 重置程序入口点，即清除镜像设置的任何默认命令

```bash
docker run -it --entrypoint="" mysql bash
```

## 开放端口及网络

- `--expose=[]`：开放容器的一个或多个端口，但是不发布到宿主机系统。
- `-P`：将所有开放的端口发布到主机接口。
- `-p=[]`：将容器的端口或端口范围发布到主机。格式：`ip:hostPort:containerPort` | `ip::containerPort` | `hostPort:containerPort` | `containerPort`。
  - `hostPort` 和 `containerPort` 都可以是一个端口范围。
  - 当两者都是端口范围时，该范围内的容器端口数必须与该范围内的主机端口数匹配，比如 `-p 1234-1236:1234-1236/tcp`。
  - 当只有 `hostPort` 是一个端口范围时，容器端口必须是指定 `hostPort` 范围内的某个端口。比如 `-p 1234-1236:1234/tcp`。
- `--link="<name or id>"`：和另一个容器建立网络连接。

**将容器的 8080 端口绑定到宿主机 127.0.0.1 的 80 端口**

```bash
docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```

## 设置环境变量

Docker 在创建 Linux 容器时会自动设置一些环境变量，但是在创建 Windows 容器时不会设置任何环境变量。下表是自动为 Linux 容器设置的环境变量：

| 变量 | 值 |
| :--------- | :--------- |
| `HOME` | Set based on the value of `USER` |
| `HOSTNAME` | The hostname associated with the container |
| `PATH` | Includes popular directories, such as `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` |
| `TERM` | `xterm` if the container is allocated a pseudo-TTY |

另外，可以使用一个或多个 `-e`、`--env`、`--env-file` 参数设置容器中的任何环境变量，甚至覆盖表中的变量，也可以由开发人员用 `Dockerfile` `ENV` 命令定义。

```bash
docker run -e MYVAR1 --env MYVAR2=foo --env-file env.list ubuntu bash

docker run --env VAR1=value1 --env VAR2=value2 ubuntu env | grep VAR
VAR1=value1
VAR2=value2
```

如果在没有指定值的情况下命名环境变量，则宿主机该变量的当前值将传播到容器的环境变量中，如果宿主机中也不存在该变量则不设置：

```bash
export today=Wednesday
docker run -e "color=purple" -e today --rm alpine env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=d2219b854598
color=purple
today=Wednesday
HOME=/root
```

设置 Windows 容器的环境变量：

```powershell
PS C:\> docker run --rm -e "id=cxfans" microsoft/nanoserver cmd /s /c set
……
id=cxfans
……
```

也可以从一个文件载入环境变量，这个文件必须使用 `<variable>=value` 的语法，或者从宿主机中取值而只给定 `<variable>`，`#` 表示注释内容。

```bash
cat env.list
# This is a comment
VAR1=value1
VAR2=value2
USER

docker run --env-file env.list ubuntu env | grep VAR
VAR1=value1
VAR2=value2
USER=denis
```

## 容器健康状况检查

```
  --health-cmd            Command to run to check health
  --health-interval       Time between running the check
  --health-retries        Consecutive failures needed to report unhealthy
  --health-timeout        Maximum time to allow one check to run
  --health-start-period   Start period for the container to initialize before starting health-retries countdown
  --no-healthcheck        Disable any container-specified HEALTHCHECK
```

**示例**

```bash
docker run --name=test -d \
    --health-cmd='stat /etc/passwd || exit 1' \
    --health-interval=2s \
    busybox sleep 1d
sleep 2; docker inspect --format='{{.State.Health.Status}}' test
healthy
docker exec test rm /etc/passwd
sleep 2; docker inspect --format='{{json .State.Health}}' test
{
  "Status": "unhealthy",
  "FailingStreak": 3,
  "Log": [
    {
      "Start": "2016-05-25T17:22:04.635478668Z",
      "End": "2016-05-25T17:22:04.7272552Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:06.732900633Z",
      "End": "2016-05-25T17:22:06.822168935Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:08.823956535Z",
      "End": "2016-05-25T17:22:08.897359124Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:10.898802931Z",
      "End": "2016-05-25T17:22:10.969631866Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:12.971033523Z",
      "End": "2016-05-25T17:22:13.082015516Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    }
  ]
}
```

## 挂载 tmpfs 文件系统

```
--tmpfs=[]: Create a tmpfs mount with: container-dir[:<options>],
            where the options are identical to the Linux
            'mount -t tmpfs -o' command.
```

```bash
docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
```

## 为容器添加数据卷

- `-v, --volume=[host-src:]container-dest[:<options>]` - 绑定一个卷，默认为读写模式。
  - `options` 以逗号分隔可以是 [rw|ro], [z|Z], [[r]shared|[r]slave|[r]private], [nocopy]。`nocopy` 模式可用于禁用自动将容器中请求的卷路径复制到卷存储位置，简而言之大概就是禁用自动复制。对于非匿名卷，`copy` 是默认模式，但绑定安装的卷不支持该模式。
  - `host-src` 是宿主机中的绝对路径或名称值，不存在则自动创建。
- `--read-only` 可以和 `-v` 联合控制容器在何处写入文件，该参数将容器的根文件系统挂载为只读，禁止写入容器指定卷以外的位置。

```bash
# 容器当前工作目录也是宿主机当前工作目录
docker  run  -v `pwd`:`pwd` -w `pwd` -i -t  ubuntu pwd

# 只允许写入指定卷
docker run --read-only -v /icanwrite busybox touch /icanwrite/here
```

- `--volumes-from=""` - 绑定来自给定容器的全部卷。
  - `:ro` 只读模式
  - `:rw` 读写模式
  - `:z` 更改标签，告诉 Docker 两个容器共享卷的内容，Docker 用共享内容的标签来标记内容，共享卷标签允许所有容器读取/写入内容。
  - `:Z` 更改标签，告诉 Docker 用私有的非共享标签来标记内容，仅当前容器可以使用私有卷。

```bash
docker run --volumes-from 777f7dc92da7 --volumes-from ba8c0c54f0f2:ro -i -t ubuntu pwd
```
像 SELinux 这样的标签系统要求在安装到容器中的卷内容上放置正确的标签。没有标签，安全系统可能会阻止容器内运行的进程使用内容。默认情况下，Docker 不会更改操作系统设置的标签。

**在容器内管理宿主机的 Docker 守护进程**

```bash
docker run -t -i -v /var/run/docker.sock:/var/run/docker.sock -v /path/to/static-docker-binary:/usr/bin/docker busybox sh
```

当用 systemd 管理 Docker 守护进程时，在 systemd 文件中有一个控制 Docker 守护进程自身挂载传播的选项，称为 `MountFlags`。这个值可能会导致 Docker 无法看到挂载点进行的挂载和传播更改。例如，当该值为 `slave` 时，则可能无法在数据卷上使用 `shared` 或者 `rshared` 传播。

**在 Windows 下路径分隔符是 `\`**

```powershell
PS C:\> docker run -v c:\foo:c:\dest microsoft/nanoserver cmd /s /c type c:\dest\somefile.txt
Contents of file

PS C:\> docker run -v c:\foo:d: microsoft/nanoserver cmd /s /c type d:\somefile.txt
Contents of file
```

以下示例在基于 Windows 的容器中将失败，因为容器内的卷或绑定挂载的目标必须是下面几种情况之一：不存在的目录、空的目录、C盘之外的驱动器。此外，绑定挂载的源必须是本地目录，而不是一个文件。

```powershell
net use z: \\remotemachine\share
docker run -v z:\foo:c:\dest ...
docker run -v \\uncpath\to\directory:c:\dest ...
docker run -v c:\foo\somefile.txt:c:\dest ...
docker run -v c:\foo:c: ...
docker run -v c:\foo:c:\existing-directory-with-contents ...
```

## USER

root (id = 0)是容器中的默认用户，镜像开发人员可以创建其他用户，可以通过名称访问这些用户，在传递数字 ID 时，用户不必存在于容器中。

开发人员可以使用 Dockerfile `USER` 指令设置一个默认用户来运行第一个进程。启动容器时，可以通过传递 `-u` 选项来覆盖 `USER` 指令。

```bash
-u="", --user="": Sets the username or UID used and optionally the groupname or GID for the specified command.

The followings examples are all valid:
--user=[ user | user:group | uid | uid:gid | user:gid | uid:group ]
```

## WORKDIR

在容器中运行二进制文件的默认工作目录是根目录（`/`），但开发人员可以使用 Dockerfile `WORKDIR` 命令设置其他默认目录，启动容器时，可以使用 `-w` 参数覆盖它：

```
docker  run -w /path/to/dir/ -it  ubuntu pwd
```

当容器内不存在路径 `/path/to/dir/` 时，会自动创建。

## 示例

## 设置容器存储驱动参数

```bash
docker create -it --storage-opt size=120G fedora /bin/bash
```

`size` 可以在创建时设置容器根文件系统大小。这个选项仅在 `overlay2`, `devicemapper`, `btrfs`, `windowsfilter`, `zfs` 文件系统中可用。对于后四个文件系统，用户不可以传入小于默认基础文件系统大小的参数；对于 `overlay2` 存储驱动，只有当其后端文件系统是 `xfs` 且以 `pquata` 参数挂载时，`size` 参数才可用。

## 设置容器元数据

标签 `-l, --label` 是将元数据应用于容器的 `key=value` 键值对，不设置 `value` 则为 `""`，可以同时设置多个标签：

```bash
docker run -l cxfans --label user=cxfans ubuntu bash
```

也可以从文件中导入多个标签，用 `EOL` 标记分隔文件中的每个标签

```bash
cat labels
com.example.label1="a label"
# this is a comment
com.example.label2=another\ label
com.example.label3

docker run --label-file labels ubuntu bash
```

## 访问 NVIDIA GPU

`--gpus` 参数允许访问 NVIDIA GPU 资源，但前提是安装了 [nvidia-container-runtime](https://nvidia.github.io/nvidia-container-runtime/)。

1. 全部 GPUs

```bash
docker run -it --rm --gpus all ubuntu nvidia-smi
```

2. 指定 GPU

```bash
docker run -it --rm --gpus device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a ubuntu nvidia-smi
```

2. 指定多个 GPU

```bash
docker run -it --rm --gpus device=0,2 nvidia-smi
```

## 添加宿主机解析

```bash
hostip=`ip -4 addr show scope global dev eth0 | grep inet | awk '{print $2}' | cut -d / -f 1`
docker run  --add-host=host:${hostip} --rm -it debian
```

## 设置容器的限额

由于在容器中设置 `ulimit` 要求特权，而默认容器是禁用特权的，因此可以使用 `--ulimit <type>=<soft limit>[:<hard limit>]` 参数进行设置。`--ulimit` 提供软限制和硬限制两种设置。

```bash
docker run --ulimit nofile=1024:1024 --rm debian sh -c "ulimit -n"
1024
```

不提供 `hard limit` 时， 默认等于 `soft limit`。`--ulimit as=` 参数目前不支持。设置好后，这些值被发送到相应的系统调用，Docker不执行任何字节转换。

小心地使用 `ulimit` 参数设置 `nproc`，因为 `nproc` 是为 Linux 设计用于设置用户而不是容器可用的最大进程数。

```bash
docker run -d -u daemon --ulimit nproc=3 busybox top
docker run -d -u daemon --ulimit nproc=3 busybox top
docker run -d -u daemon --ulimit nproc=3 busybox top

# 失败
docker run -d -u daemon --ulimit nproc=3 busybox top
```

## 设置容器退出信号

`--stop-signal` 可以设置用于容器退出的系统调用信号，该信号可以是与内核 syscall 表中的位置匹配的有效无符号数字，也可以是 SIGNAME 格式的信号名称。

## 超时停止容器

`--stop-timeout` 参数设置将发送到容器退出的预定义系统调用信号的超时（以秒为单位）。超时后，该容器将被 `SIGKILL` 杀死。

## 指定容器的隔离技术

`--isolation=<value>` 参数可以指定容器的隔离技术。在 Linux 上，唯一支持 Linux 命名空间的默认选项。因此以下两个命令是等价的。

```bash
docker run -d busybox top
docker run -d --isolation default busybox top
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

## 配置命名空间的内核参数

`--sysctl` 可在容器中设置命名空间的内核参数 `sysctls`，但并非所有 `sysctls` 都有名称空间，Docker 不支持更改会破坏宿主机系统的 `sysctls`。

### 在容器网络命名空间中打开 IP 转发

```bash
docker run --sysctl net.ipv4.ip_forward=1 centos
```

### 当前支持的 `sysctls`

- IPC Namespace

```
kernel.msgmax, kernel.msgmnb, kernel.msgmni, kernel.sem, kernel.shmall, kernel.shmmax, kernel.shmmni, kernel.shm_rmid_forced
Sysctls beginning with fs.mqueue.*
```

- Network Namespace

```
net.*
```

命名空间为 `host` 模式时，无法工作。
