# 浅析 Runtime 标准

> [Runtime](https://github.com/opencontainers/runtime-spec) 标准由 Open Container Initiative（OCI）负责定义维护，[runc](https://github.com/opencontainers/runc) 就是 Docker 公司贡献给 OCI 的一个标准 runtime 实现。

## 规范内容

在 Linux 平台上，跟 runtime 有关的规范大概有五个，分别是：

- Filesystem Bundle: [bundle.md](https://github.com/opencontainers/runtime-spec/blob/master/bundle.md)
- Container Configuration File: [config.md](https://github.com/opencontainers/runtime-spec/blob/master/config.md)
- Runtime and Lifecycle: [runtime.md](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md)
- Linux Container Configuration: [config-linux.md](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md)
- Linux Runtime: [runtime-linux.md](https://github.com/opencontainers/runtime-spec/blob/master/runtime-linux.md)

## Filesystem Bundle

[上一篇笔记](runc.md)中制作了一个符合 OCI 标准的 bundle 文件夹，以之为例，其 bundle 的目录结构是这样的：

```bash
$ tree -L 2 bundle
bundle
├── config.json
└── rootfs
    ├── dev
    ├── etc
    ├── hello
    ├── proc
    └── sys
```

有了这个 bundle 后，符合 runtime 标准的程序就可以根据 bundle 启动容器了。标准的 bundle 由两部分组成：

- `config.json` 即下文的 Container Configuration File，这个文件的名称是规定的，不可以改名。
- **容器的根文件系统目录**  一般命名为 rootfs。实际中，根文件系统目录可以在其它地方，只需在 config.json 中配置正确的路径就可以了。

## Container Configuration File

该规范定义了 `config.json` 文件，该文件的内容至少也在一百多行，所以这里不列出了，可以参看[官方内容示例](https://github.com/opencontainers/runtime-spec/blob/master/config.md#configuration-schema-example)。

这里只简单介绍几个字段：

- root（必须有）：根文件系统的路径。
- mounts：挂载到容器的目录。Linux 平台必须包含 `/proc`, `/sys`, `/dev/pts`, `/dev/shm` 这四个目录。
- [process](https://github.com/opencontainers/runtime-spec/blob/master/config.md#process)：容器启动后执行的初始命令。
- hostname：容器的主机名。
- linux：Linux 平台的特殊配置，即下文的 `Linux Container Configuration`。

## Linux Container Configuration

[该规范](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md)是 Linux 平台上对 Container Configuration File 的扩展，这部分的内容也包含在 `config.json` 文件中。

### 命名空间相关配置

1. 参数

- **`type`** *(string, REQUIRED)*，命名空间类型，支持 `pid`, `network`, `mount`, `ipc`, `uts`, `user`, `cgroup` 七种类型。

- **`path`** *(string, OPTIONAL)*，命名空间文件路径，必须是容器 Mount 命名空间中的绝对路径。不指定则创建新类型的命名空间。

2. 示例

```json
"namespaces": [
    {
        "type": "pid",
        "path": "/proc/1234/ns/pid"
    },
    {
        "type": "network",
        "path": "/var/run/netns/neta"
    },
    {
        "type": "cgroup"
    }
]
```

### User 命名空间映射

1. 字段

- **`uidMappings`** *(array of objects, OPTIONAL)*，主机和容器用户之间的映射关系。
- **`gidMappings`** *(array of objects, OPTIONAL)*，主机和容器用户组之间的映射关系。

2. 参数

- **`containerID`** *(uint32, REQUIRED)* - 容器中 `uid/gid` 的起始序号。
- **`hostID`** *(uint32, REQUIRED)* - 宿主机中与 `containerID`  映射的 `uid/gid` 的起始序号。
- **`size`** *(uint32, REQUIRED)* - 被映射 `uid/gid` 的数量。

3. 示例

```json
"uidMappings": [
    {
        "containerID": 0,
        "hostID": 1000,
        "size": 32000
    }
],
"gidMappings": [
    {
        "containerID": 0,
        "hostID": 1000,
        "size": 32000
    }
```

### 设备权限

1. 字段

- **`devices`** *(array of objects, OPTIONAL)* ：设置哪些设备可以在容器内被访问到。以下这些设备默认就能在容器内访问到，即 runtime 的实现需要默认将这些设备绑定到容器内：

    - `/dev/null`
    - `/dev/zero`
    - `/dev/full`
    - `/dev/random`, `/dev/urandom`
    - `/dev/tty`
    - `/dev/console` - 在 `process` 中配置启动 `terminal` 选项时必须挂载
    - `/dev/ptmx` - `/dev/pts/ptmx` 的绑定挂载或链接

2. 参数

- **`type`** *(string, REQUIRED)* - 设备类型: `c`, `b`, `u` 或 `p`。
- **`path`** *(string, REQUIRED)* - 容器内设备的绝对路径。
- **`major, minor`** *(int64, REQUIRED unless `type` is `p`)* - 设备的 [major, minor numbers](https://www.kernel.org/doc/Documentation/admin-guide/devices.txt)。
- **`fileMode`** *(uint32, OPTIONAL)* - 打开模式。
- **`uid`** *(uint32, OPTIONAL)* - 容器命名空间中该设备所有者的 `uid`。
- **`gid`** *(uint32, OPTIONAL)* - 容器命名空间中该设备所有者的 `gid`。

3. 示例

```json
"devices": [
    {
        "path": "/dev/fuse",
        "type": "c",
        "major": 10,
        "minor": 229,
        "fileMode": 438,
        "uid": 0,
        "gid": 0
    },
    {
        "path": "/dev/sda",
        "type": "b",
        "major": 8,
        "minor": 0,
        "fileMode": 432,
        "uid": 0,
        "gid": 0
    }
]
```

### 控制组相关配置

#### 控制组路径

1. 字段

**`cgroupsPath`** *(string, OPTIONAL)* ：控制组的路径，可以是绝对路径或相对路径，但这个绝对路径是相对于容器中 cgroup 挂载点的绝对路径，相对路径则可能被解释为相对于 runtime 指定的 cgroups hierarchy 的位置。

2. 示例

```json
"cgroupsPath": "/Runtime/Container",
"resources": {
    "memory": {
    "limit": 100000,
    "reservation": 200000
    },
    "devices": [
        {
            "allow": false,
            "access": "rwm"
        }
    ]
}
```

#### 设备白名单

1. 字段

**`devices`** *(array of objects, OPTIONAL)* ：配置设备白名单，runtime 必须按列表顺序申请访问。

2. 参数

- **`allow`** *(boolean, REQUIRED)* - 是否允许访问。
- **`type`** *(string, OPTIONAL)* - 设备类型：`a` (all), `c` (char), or `b` (block)。默认 `a`。
- **`major, minor`** *(int64, OPTIONAL)* - 设备的 [major, minor numbers](https://www.kernel.org/doc/Documentation/admin-guide/devices.txt)。默认 "all", 即 [`*`](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt)。
- **`access`** *(string, OPTIONAL)* - 设备的 cgroup 权限。可以是 `r` (`read`), `w` (`write`), `m` (`mknod`) 的组合。

3. 示例

```json
"resources": {
    "devices": [
        {
            "allow": false,
            "access": "rwm"
        },
        {
            "allow": true,
            "type": "c",
            "major": 10,
            "minor": 229,
            "access": "rw"
        },
        {
            "allow": true,
            "type": "b",
            "major": 8,
            "minor": 0,
            "access": "r"
        }
    ]
}
```

#### 内存限制

1. 字段

**`memory`** *(objects, OPTIONAL)* ：Cgroup 内存子系统的配置。

2. 参数

- **`limit`** *(int64, OPTIONAL)* - 内存上限。
- **`reservation`** *(int64, OPTIONAL)* - 非强制内存上限。
- **`swap`** *(int64, OPTIONAL)* - 内存和交换空间上限。
- **`kernel`** *(int64, OPTIONAL)* - 强制 Kernel 内存上限。
- **`kernelTCP`** *(int64, OPTIONAL)* - 强制 Kernel TCP 缓冲内存上限。
- **`swappiness`** *(uint64, OPTIONAL)* - vmscan 的 swappiness 参数。值为 0 到 100 之间。
- **`disableOOMKiller`** *(bool, OPTIONAL)* - 关闭或开启 OOM killer。
- **`useHierarchy`** *(bool, OPTIONAL)* - 关闭或开启全局内存限制。开启（`true`），则子 cgroups 共享此 cgroup 的内存限制。

3. 示例

```json
"resources": {
    "memory": {
        "limit": 536870912,
        "reservation": 536870912,
        "swap": 536870912,
        "kernel": -1,
        "kernelTCP": -1,
        "swappiness": 0,
        "disableOOMKiller": false
    }
}
```

#### CPU 限制

1. 字段

**`cpu`** *(objects, OPTIONAL)* ：Cgroup cpu 和 cpusets 子系统的配置。

2. 参数

- **`shares`** *(uint64, OPTIONAL)* - 指定 CPU 相对占用率。
- **`quota`** *(int64, OPTIONAL)* - 指定一个周期中 CPU 的占用时间（微秒）。
- **`period`** *(uint64, OPTIONAL)* - 指定一个周期的时长（微秒），仅适用于 CFS 调度算法。
- **`realtimeRuntime`** *(int64, OPTIONAL)* - 指定可以访问 CPU 的最长连续时间。
- **`realtimePeriod`** *(uint64, OPTIONAL)* - 与 **`period`** 相同，但仅适用于实时调度算法。
- **`cpus`** *(string, OPTIONAL)* - 容器可用 CPUs 列表。
- **`mems`** *(string, OPTIONAL)* - 容器可用内存节点列表。

3. 示例

```json
"resources": {
    "cpu": {
        "shares": 1024,
        "quota": 1000000,
        "period": 500000,
        "realtimeRuntime": 950000,
        "realtimePeriod": 1000000,
        "cpus": "2-3",
        "mems": "0-7"
    }
}
```

#### 网络

1. 字段

**`network`** *(objects, OPTIONAL)* ：Cgroup net_cls 和 net_prio 子系统的配置。

2. 参数

- **`classID`** *(uint32, OPTIONAL)* - cgroup 中网络数据包将使用的网络类标识符。

- **`priorities`** *(array of objects, OPTIONAL)* - 指定一组优先级的对象列表，这些优先级对象分配给源自组中进程并在各种接口上流出系统的流量。 可以根据优先级指定以下参数：
  - **`name`** *(string, REQUIRED)* - 接口名
  - **`priority`** *(uint32, REQUIRED)* - 接口优先级

3. 示例

```json
"resources": {
    "network": {
        "classID": 1048577,
        "priorities": [
            {
                "name": "eth0",
                "priority": 500
            },
            {
                "name": "eth1",
                "priority": 1000
            }
        ]
   }
}
```


#### 进程 PIDs

1. 字段

- **`pids`** *(objects, OPTIONAL)* ：Cgroup pids 子系统的配置。

2. 参数

- **`limit`** *(uint32, OPTIONAL)* - 指定 cgroup 中进程号的最大值。

3. 示例

```json
"resources": {
    "pids": {
        "limit": 32771
   }
}
```

#### 其他控制组字段

不常用，暂略。

### 根文件系统挂载类型

1. 参数值

- **`rootfsPropagation`** *(string, OPTIONAL)* 可以是 `slave`, `private`, `shared` 或 `unbindable`。

2. 示例

```json
    "rootfsPropagation": "slave",
```

### 用户不可见目录

1. 参数

- **`rootfsPropagation`** *(array of strings, OPTIONAL)* 设置容器内的哪些目录对用户不可见，必须是命名空间的绝对路径。

2. 示例

```json
    "maskedPaths": [
        "/proc/kcore"
    ]
```

### 只读目录

1. 参数

- **`readonlyPaths`** *(array of strings, OPTIONAL)* 设置容器内的哪些目录是只读的。

2. 示例

```json
    "readonlyPaths": [
        "/proc/sys"
    ]
```

### 其他字段

不常用，暂略。

## Runtime and Lifecycle

[该规范](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md)定义了跟容器运行时相关的三部分内容，容器的状态、容器相关的操作以及容器的生命周期。

### 容器的状态

1. 示例

```json
{
    "ociVersion": "0.2.0",
    "id": "oci-container1",
    "status": "running",
    "pid": 4422,
    "bundle": "/containers/redis",
    "annotations": {
        "myKey": "myValue"
    }
}
```

2. 字段解析

- **`ociVersion`** (string, REQUIRED)
- **`id`** (string, REQUIRED) 容器 ID，必须唯一。
- **`status`** (string, REQUIRED) 容器运行状态，可以是：
    - `creating`: 正在被创建（生命周期第二阶段）。
    - `created`: 已创建（生命周期第二阶段之后）。
    - `running`: 正在运行（生命周期第五阶段之后）。
    - `stopped`: 已停止运行（生命周期第七阶段）。
- **`pid`** (int, REQUIRED 容器内第一个进程在系统初始 PID 命名空间中的 PID，即在容器宿主机看到的 PID。
- **`bundle`** (string, REQUIRED) bundle 的绝对路径。
- **`annotations`** (map, OPTIONAL) 容器的注释，相当于容器标签。

### 容器相关的操作

该部分定义了一个符合标准的实现必须支持这些[命令](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md#operations)：

- `state <container-id>` 查询容器状态
- `create <container-id> <path-to-bundle>` 创建容器
- `start <container-id>` 启动容器，执行 config.json 中 process 部分指定的进程
- `kill <container-id> <signal>` 终止容器
- `delete <container-id>` 删除容器

### 容器的生命周期

[生命周期](https://github.com/opencontainers/runtime-spec/blob/master/runtime.md#lifecycle)描述了从容器被创建到被删除的时间线。

1. 执行 `create <container-id> <path-to-bundle>` 创建容器，相关参数将指定容器的唯一标识符和 bundle 的路径。
2. 根据 config.json 中的配置创建容器运行时必需的环境和资源，但此时不运行用户（process）指定的程序，这步执行完成之后，表示容器创建成功。此后，对 config.json 的任何更新均不影响容器。
3. 执行 `start <container-id>` 根据唯一标识符启动容器。
4. 执行预启动钩子（prestart hooks）。启动失败将终止容器，然后继续生命周期第九阶段。
5. 执行 config.json 中 process 部分指定的进程。
6. 执行 [poststart](https://github.com/opencontainers/runtime-spec/blob/master/config.md#poststart) 钩子。执行失败只记录警告，仍将继续。
7. 容器退出。
8. 执行 `delete <container-id>` 根据唯一标识符删除容器。
9. 必须通过取消在创建阶段（第二阶段）执行的步骤来销毁容器。
10. 执行 [poststop](https://github.com/opencontainers/runtime-spec/blob/master/config.md#poststop) 钩子。执行失败只记录警告，跟执行成功无区别。

## Linux Runtime

声明了创建容器时（生命周期第二阶段）在源文件存在的情况下处理完 `config.json` 中 `mounts` 字段之后必须建立的软链接：

| Source | Destination |
| ----------- | ----------- |
| /proc/self/fd | /dev/fd |
| /proc/self/fd/0 | /dev/stdin |
| /proc/self/fd/1 | /dev/stdout |
| /proc/self/fd/2 | /dev/stderr |

## 结语

Docker 负责准备 runtime 的 bundle 以及网络、存储等外在资源，而 runtime 负责运行该容器，然后管理其生命周期。

### 章节导航

- 上一节：[通过 runc 运行容器](runc.md)
- 下一节：暂无
