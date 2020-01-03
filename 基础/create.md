# 浅析 CREATE 执行过程

> `docker run` 命令直接创建并运行了一个容器，它的背后其实包含独立的两步，一步是 `docker create` 创建容器，另一步是 `docker start` 启动容器。

这个执行过程，简而言之，dockerd 在收到 docker 创建容器的请求后，做了两件事情：一是**准备容器的 layer**，二是检查传过来的参数，然后和**镜像配置文件**中的参数进行合并，最后存储成**容器的配置文件**。

## 创建实验容器

1. 为了防止干扰，清空已有的容器

```bash
$ docker rm $(docker ps -aq)
```

2. 创建演示容器

```bash
$ docker create -it --name demo ubuntu
6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
```

- `-i` 保持标准输入开启，这里就是打开终端交互模式，在创建时意义不大。
- `-t` 分配一个伪 tty。Ubuntu 默认启动进程是 bash，而 bash 需要 tty，否则会异常退出。

3. 显示当前容器状态

```bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
6e7cf3dd81db        ubuntu              "/bin/bash"         32 seconds ago      Created                                 demo
```

## 容器 layer 的元数据

创建容器时，Docker 会为每个容器在 `/var/lib/docker/<storage-driver>` 目录中创建两个新的 layer，一个是只读的 `init layer`，包含 Docker 为容器准备的一些文件，另一个是可写的 `mount layer`，在容器中对 rootfs 的所有增删改操作的结果都会存在这个 layer 中。

容器 layer 的元数据都存储在 `image/overlay2/layerdb/mounts` 目录下（当前的工作目录为 `/var/lib/docker`），目录名称就是容器 ID。这篇笔记特别容易和上一篇笔记[《简述 Image 的获取和存储方式》](pull.md)混淆，而且不少地方遥相呼应，然后就绕晕了。

```bash
# 路径过长，有所省略
$ ls 6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
init-id  mount-id  parent

$ cat mount-id
29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493

$ cat init-id
29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493-init

$ cat parent
sha256:7c52cdc1e32d67e3d5d9f83c95ebe18a58857e68bb6985b0381ebdcec73ff303
```

- `mount-id` 文件包含了 `mount layer` 的 `cacheid`。
- `init-id` 文件包含了 `init layer` 的 `cacheid`，就是在 `mount layer` 的 `cacheid` 后面加上了一个 `-init`。
- `parent` 文件包含的是**镜像**（不是容器，这里只是元数据而已）最顶层 layer 的 `chainid`，表示这个容器的 `init layer` 的父 layer。可以根据 `parent` 的 `chainid` 得到 `parent` 的 `diffid`，

```bash
$ cat image/overlay2/layerdb/sha256/7c52cdc1e32d67e3d5d9f83c95ebe18a58857e68bb6985b0381ebdcec73ff303/diff
sha256:918efb8f161b4cbfa560e00e8e0efb737d7a8b00bf91bb77976257cd0014b765
```

新加的这两层 layer 比较特殊，这两层的元数据只保存在 `layerdb/mounts` 下面，在 `layerdb/sha256` 目录下没有相关信息，说明 Docker 将**容器的 layer 和镜像的 layer 的元数据**放在了不同的两个目录中。

## 容器 layer 数据

容器 layer 的数据和镜像 layer 的数据的管理方式是一样的，都存在 `/var/lib/docker/<storage-driver>` 目录中。因为当时不知道两者区别，就一起写在了[《简述 Image 的获取和存储方式》](pull.md)这篇笔记里，所以这里不再重复，只更深入了解一些之前忽略的细节。

```bash
# 前面 `cat init-id` 获得的 cacheid
$ tree 29cb9542bd94b7be614f983a6984f9cb136b8591b14c1a601d656a5fe0e3f493-init/diff
diff
├── dev
│   ├── console
│   ├── pts
│   └── shm
└── etc
    ├── hostname
    ├── hosts
    ├── mtab -> /proc/mounts
    └── resolv.conf
```

这些文件都是 Linux 运行时必须的文件，缺少或将引起异常。

- `/dev/console`：该文件一般指向主机的当前控制台，有些程序会依赖该文件。在容器启动时，Docker 会为容器创建一个 `pts`，然后通过 `bind mount` 的方式将 `pts` 绑定到容器里面的 `/dev/console` 上，这样在容器中往这个文件里面写数据就相当于往容器的控制台上打印数据。这里预先创建一个空文件相当于占位符，作为后续 `bind mount` 的目的路径。

- `/dev/pts`：[pseudo-terminals](https://en.wikipedia.org/wiki/Pseudo_terminal) 的缩写，翻译过来就是伪终端，暂且简单地认为是 tty 设备目录即可，实际上只存在于内存。

- `/dev/shm`：是一个用于临时文件存储的内存文件系统，通常是 tmpfs。

- `hostname`，`hosts`，`resolv.conf`：对于每个容器来说，容器内的这几个文件内容可能都不一样，这里也只是占位，等 Docker 在外面生成这几个文件，然后通过 `bind mount` 的方式将这些文件绑定到容器中的这些位置，即这些文件都会被宿主机中的文件覆盖掉。

- `/etc/mtab`：这个文件在新的 Linux 发行版中都指向 `/proc/mounts`，里面包含了当前 Mount 命名空间中的所有挂载信息，很多程序和库会依赖这个文件。

## 配置文件

Docker 将用户指定的参数和镜像配置文件中的部分参数进行合并，然后将合并后生成的容器的配置文件放在 `/var/lib/docker/containers/` 下，目录名称就是容器 ID。

```bash
$ cd 6e7cf3dd81db2c50649a62897a83382d35e6fc075847d860b2ec29d5ff4c030b
$ tree
.
├── checkpoints
├── config.v2.json
└── hostconfig.json

1 directory, 2 files
```

- `config.v2.json`：通用的配置。
- `hostconfig.json`：主机相关的配置。
- `checkpoints`：可以在当前节点做一个 `checkpoint`，然后再到另一个节点上继续运行，相当于无缝的将一个正在运行的进程先暂停，然后迁移到另一个节点上继续运行。

### 章节导航

- 上一节：[简述 Image 的获取和存储方式](pull.md)
- 下一节：[浅析 START 执行过程](start.md)
