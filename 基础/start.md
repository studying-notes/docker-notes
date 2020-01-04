# 浅析 START 执行过程

> `docker run` 命令直接创建并运行了一个容器，它的背后其实包含独立的两步，一步是 `docker create` 创建容器，另一步是 `docker start` 启动容器。

## 启动实验容器

1. 启动前一篇笔记中创建的容器

```bash
$ docker start demo
demo
```

2. 显示当前容器的状态

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6e7cf3dd81db        ubuntu              "/bin/bash"         3 hours ago         Up 2 minutes                            demo
```

## 容器启动的大致流程

1. docker 发送启动容器命令给 dockerd；
2. dockerd 收到请求后，准备好 rootfs，以及一些其他的配置文件，然后通过 gRPC 的方式通知 containerd 启动容器；
3. containerd 根据收到的请求以及配置文件位置，创建容器运行时所需的 bundle，然后启动 shim 进程，让它来启动容器；
4. shim 进程启动后，做一些准备工作，然后调用 runc 启动容器。

## dockerd 行为详解

1. 准备 rootfs

在 `docker create` 创建容器之后，容器的所有 layer 都已经准备好了，现在就差一步将它们都合并起来，对于 Overlay2，需要通过 `mount` 的方式将所有的 layer 联合挂载起来。这部分在[之前的笔记](pull.md)里就有所涉及。

```bash
$ ls overlay2/29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493/merged
bin/   dev/  home/  lib64/  mnt/  proc/  run/   srv/  tmp/  var/
boot/  etc/  lib/   media/  opt/  root/  sbin/  sys/  usr/
```

- `merged`：OverlayFS 的 `merged` 文件夹，即联合挂载点所在的目录，包含了所有 layer 的内容（即 `diff` 文件夹中的项目），最顶层的 layer 的 `diff` 文件夹作为 OverlayFS 的 `upperdir`，其余 layer 的 `diff` 文件夹依层次顺序作为 OverlayFS 的 `lowerdir`。

2. 准备容器内部配置文件

```bash
$ cd containers/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ tree
.
├── 6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b-json.log
├── checkpoints
├── config.v2.json
├── hostconfig.json
├── hostname
├── hosts
├── mounts
├── resolv.conf
└── resolv.conf.hash

2 directories, 7 files
```

与创建时相比，多了几个文件，这几个文件都是 Docker 动态生成的：

- `6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b-json.log`：容器的日志文件，后续容器的 `stdout` 和 `stderr` 都会输出到这个目录，除非另外安装配置了日志插件。
- `hosts`：容器的 `hosts` 文件，一般根据 Docker 的网络配置生成。
- `hostname`：容器的主机名，来自于 `config.v2.json` 文件，由 `docker create` 命令的 `-h` 参数指定，默认为容器 ID 的前 12 位。
- `mounts`：挂载目录。
- `resolv.conf`：DNS 服务器，来自于 `hostconfig.json`，由 `docker create` 命令的 `--dns` 参数指定，默认与宿主机配置相同或者是 Docker 桥接网络的 IP。
- `resolv.conf.hash`：`resolv.conf` 文件的校验码。

除了日志文件外，其他文件在每次容器启动的时候都会重新自动生成，所以修改它们的内容后只会在当前容器运行的时候生效，容器重启后，配置又都会恢复到默认生成的状态。

3. 准备 OCI Bundle

dockerd 在生成这个文件前，必须创建好 cgroup 的相关目录、准备网络相关的配置等，然后才生成 config.json 文件。Cgroup 相关目录：

```bash
$ find /sys/fs/cgroup -name 6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/freezer/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/devices/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/perf_event/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/pids/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/blkio/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/net_cls,net_prio/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/hugetlb/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/cpuset/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/cpu,cpuacct/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/memory/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
/sys/fs/cgroup/systemd/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
```

Bundle 文件被放在 `/run/containerd/io.containerd.runtime.v1.linux/moby/<containerd-id>` 目录下，只有当容器在运行的时候，该目录才存在，容器终止后该目录被删除掉，下一次启动时会再次被创建。

```bash
$ cd /run/containerd/io.containerd.runtime.v1.linux/moby/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ tree
.
├── config.json
├── init.pid
├── log.json
└── rootfs
```

其中与 Bundle 相关的其实只有 config.json 文件，内容有几千行，这里只截取部分：

```json
{
    "root": {
        "path": "/var/lib/docker/overlay2/29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493/merged"
    },
    "mounts": [
        {
            "destination": "/sys/fs/cgroup",
            "type": "cgroup",
            "source": "cgroup",
            "options": [
                "ro",
                "nosuid",
                "noexec",
                "nodev"
            ]
        },
        {
            "destination": "/dev/shm",
            "type": "tmpfs",
            "source": "shm",
            "options": [
                "nosuid",
                "noexec",
                "nodev",
                "mode=1777",
                "size=67108864"
            ]
        },
        {
            "destination": "/etc/resolv.conf",
            "type": "bind",
            "source": "/var/lib/docker/containers/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b/resolv.conf",
            "options": [
                "rbind",
                "rprivate"
            ]
        },
        {
            "destination": "/etc/hostname",
            "type": "bind",
            "source": "/var/lib/docker/containers/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b/hostname",
            "options": [
                "rbind",
                "rprivate"
            ]
        },
        {
            "destination": "/etc/hosts",
            "type": "bind",
            "source": "/var/lib/docker/containers/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b/hosts",
            "options": [
                "rbind",
                "rprivate"
            ]
        }
    ],
    "linux": {
        "cgroupsPath": "/docker/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b",
    },
}
```

4. 准备 IO 文件

dockerd 创建了一些跟 IO 相关的命名管道，用于容器之间的通信，这些文件位于 `/run/docker/containerd/<containerd-id>` 目录下。

```bash
$ cd /run/docker/containerd/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ tree
.
├── init-stdin
└── init-stdout

0 directories, 2 files

# 这两个文件是命名管道文件
$ file init-stdin init-stdout
init-stdin:  fifo (named pipe)
init-stdout: fifo (named pipe)

# 它们被 dockerd 和 docker-containerd-shim 两个进程所打开
$ lsof *
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
container   497 root   12u  FIFO   0,24      0t0 1646 init-stdin
container   497 root   13u  FIFO   0,24      0t0 1646 init-stdin
container   497 root   15r  FIFO   0,24      0t0 1646 init-stdin
container   497 root   16u  FIFO   0,24      0t0 1647 init-stdout
container   497 root   17w  FIFO   0,24      0t0 1647 init-stdout
container   497 root   18u  FIFO   0,24      0t0 1647 init-stdout
container   497 root   19r  FIFO   0,24      0t0 1647 init-stdout
container   497 root   20w  FIFO   0,24      0t0 1646 init-stdin
dockerd   28188 root   22u  FIFO   0,24      0t0 1646 init-stdin
dockerd   28188 root   23u  FIFO   0,24      0t0 1647 init-stdout
dockerd   28188 root   25w  FIFO   0,24      0t0 1646 init-stdin
dockerd   28188 root   26r  FIFO   0,24      0t0 1647 init-stdout
```

`container` 就是 `containerd-shim` 进程。上面只有 `init-stdin` 和 `init-stdout`，没有 `init-stderr`，是因为创建容器时指定了 `-t` 参数，意思是让 Docker 为容器创建一个 `tty`（虚拟的），在这种情况下，`stdout` 和 `stderr` 将采用同样的通道，即容器中进程往 `stderr` 中输出数据时，会写到 `init-stdout` 中。

这些文件都准备好了之后，dockerd 通过 gRPC 的方式给 containerd 发送请求，通知 containerd 启动容器。

## containerd 行为详解

containerd 的功能是启动容器、管理运行中的所有容器。

![containerd](../插图/containerd.png)

1. 准备相关文件

和 Bundle 文件一起被放在了 `/run/containerd/io.containerd.runtime.v1.linux/moby/<containerd-id>` 目录下，之前已经看过了目录结构。

```bash
$ cd /run/containerd/io.containerd.runtime.v1.linux/moby/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ file *
config.json: ASCII text, with very long lines, with no line terminators
init.pid:    ASCII text, with no line terminators
log.json:    empty
rootfs:      directory

$ cat init.pid
1678
```

- `log.json`：写入 runc 运行失败时的日志。
- `init.pid`：容器启动后，runc 将容器中第一个进程在容器外看到的 PID 写到这个文件中。

2. 启动过程

不知道，暂略。

3. 监听容器

待容器启动之后，containerd 还需监听容器的 OOM 事件和容器退出事件，以便及时作出响应，OOM 事件通过 Cgroup 的内存限制机制进行监听（通过 `cgroup.event_control`），而容器退出事件通过 `exit` 这个命名 `pipe` 来实现。

## shim 行为详解

shim 进程被 containerd 启动后，第一步是设置子孙进程成为孤儿进程后由 shim 进程接管，即 shim 将变成孤儿进程的父进程，这样就保证容器里的第一个进程不会因为 runc 进程的退出而被 systemd 进程接管。

接着根据传入的参数设置好要启动进程的 `stdin`，`stdout`，`stderr`（`init-stdin`，`init-stdout`，`init-stderr`），然后调用 `runc create` 命令创建容器，容器创建成功后，runc 会将容器的第一个进程的 PID 写入上面 containerd 目录下的 `init.pid` 文件中，这样containerd 进程就知道容器创建成功了，于是 containerd 接着就会调用 `runc start` 启动容器。

## runc 行为详解

runc 会被调用两次，第一次是 shim 调用 `runc create` 创建容器，第二次是 containerd 调用 `runc start` 启动容器。

1. 创建容器

runc 根据参数中传入的 bundle 目录名称以及容器 ID，创建容器。

创建容器就是启动进程 `/proc/self/exe init`，由于 `/proc/self/exe` 指向的是自己，所以相当于 fork 了一个新进程，并且新进程启动的参数是 `init`，相当于运行了 `runc init`，`runc init` 会根据配置创建好相应的命名空间，同时创建一个叫 `exec.fifo` 的临时文件，等待其他进程打开这个文件，如果有其他进程打开这个文件，则启动容器。

2. 启动容器

启动容器就是运行 `runc start`，它会打开读取文件 `exec.fifo`，从而触发 `runc init` 进程启动容器，如果读取该文件没有异常，则将会删掉文件 `exec.fifo`，所以一般情况下我们看不到文件 `exec.fifo`。

runc 创建的容器都会在 `/run/docker/runtime-runc/moby/<containerd-id>` 下有一个目录，里面有一个 `state.json` 文件（`exec.fifo` 这个临时文件也在这里），包含当前容器详细的配置及状态信息。

```bash
$ cd /run/docker/runtime-runc/moby/6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ file *
state.json: ASCII text
```

该文件内容几千行，这里仅截取部分展示：

```json
{
    "id": "6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b",
    "init_process_pid": 1678,
    "init_process_start": 16955,
    "created": "2019-12-21T11:22:23.591306172Z",
    "config": {
        "no_pivot_root": false,
        "parent_death_signal": 0,
        "rootfs": "/var/lib/docker/overlay2/29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493/merged",
        "readonlyfs": false,
    }
}
```

### 章节导航

- 上一节：[浅析 CREATE 执行过程](create.md)
- 下一节：
