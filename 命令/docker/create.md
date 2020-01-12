# `docker create` 命令详解

> 创建一个新的容器

## 基本信息

### 用法

```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### 选项

参数选项因为太多，另写成了单独的笔记，见[《`docker create` 参数选项》](create-opts.md)。

## 扩展信息

`docker create` 命令在指定的镜像之上创建了一个可写的容器层，然后为运行指定的命令做好准备。最后在 STDOUT 中打印容器 ID。这和 `docker run -d` 的结果相似，除了容器未被启动。之后可以在任何时候运行 `docker start <container_id>` 命令启动容器。

## 示例

### 创建然后启动一个容器

```bash
$ docker create -it fedora bash
fe5557cd6a912158adf3f2de2d1c52cbcbdaed7660bc064cd5d9f59b3f303a3a

$ docker start -ai fe5557cd6a
```

### 初始化数据卷

从 v1.4.0 开始，容器数据卷在 `docker create` 阶段被初始化。比如，你可以 `docker create` 一个数据卷容器，然后在另一个容器中使用。

```bash
$ docker create -v /data --name data ubuntu
321b73f209ff56bd8c638a553a46c4b15791647780250ceb4e5b426443a4308d

$ docker run --rm --volumes-from data ubuntu ls -la /data
total 8
drwxr-xr-x 2 root root 4096 Dec 27 04:15 .
drwxr-xr-x 1 root root 4096 Dec 27 04:15 ..
```

类似的，`docker create` 一个主机目录绑定挂载的数据卷容器，可以在后续容器中使用它：

```bash
$ docker create -v /root/config:/docker --name docker ubuntu
4cd4b31b062af47e0a9eb2632bd0d859c2aed87a40521bf1b95f103991adc5ac

$ docker run --rm --volumes-from docker ubuntu ls -la /docker
total 12
drwxr-xr-x 2 root root 4096 Dec 23 06:22 .
drwxr-xr-x 1 root root 4096 Dec 27 04:21 ..
-rw-r--r-- 1 root root 1065 Dec 23 06:23 config.json
```

### 设置容器存储驱动参数

```bash
$ docker create -it --storage-opt size=120G fedora /bin/bash
```

`size` 可以在创建时设置容器根文件系统大小。这个选项仅在 `overlay2`, `devicemapper`, `btrfs`, `windowsfilter`, `zfs` 文件系统中可用。对于后四个文件系统，用户不可以传入小于默认基础文件系统大小的参数；对于 `overlay2` 存储驱动，只有当其后端文件系统是 `xfs` 且以 `pquata` 参数挂载时，`size` 参数才可用。

### 指定容器的隔离技术

> 参考 [docker build](build.md#指定容器的隔离技术) 命令，这部分内容是相同的。

`--isolation=<value>` 参数可以指定容器的隔离技术。在 Linux 上，唯一支持 Linux 命名空间的默认选项。在 Windows 上，可以指定以下值:

| 值 | 描述 |
| :-------- | :----------------------------------------------------------- |
| `default` | Use the value specified by the Docker daemon’s `--exec-opt` . If the `daemon` does not specify an isolation technology, Microsoft Windows uses `process` as its default value. |
| `process` | Namespace isolation only. |
| `hyperv` | Hyper-V hypervisor partition-based isolation. |

### 处理动态创建的设备

容器可访问的设备在创建时被分配，一旦启动容器，分配的设备都将添加到 `cgroup.allow` 文件中。于是当将新设备添加到正在运行的容器时，会带来问题。

解决方案之一是向容器添加更宽松的规则，以允许其访问更广泛的设备。例如，假设容器需要访问主设备 `42` 和任意数量的副设备（随着新设备的出现而添加），那么可以添加以下规则:

```bash
docker create --device-cgroup-rule='c 42:* rmw' -name my-container my-image
```

然后可以通过 `udev` 在添加设备时执行 `docker exec my-container mknod newDevX c 42 <minor>`。
