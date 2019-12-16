# Mount 命名空间

> Mount 命名空间用于隔离各个进程的挂载点视图，在不同的 Mount 命名空间的进程中，看到的文件系统层次是不一样的，相关操作也不影响全局的文件系统。

## 概述

当前进程所在 Mount 命名空间里的所有挂载信息可以在以下三个文件里找到：

```
/proc/[pid]/mounts
/proc/[pid]/mountinfo
/proc/[pid]/mountstats
```

## Go 语言实现 Mount 命名空间隔离

```go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

## 实验过程

1. 执行 `go run mount.go` 命令

```bash
$ go run pid.go
readlink /proc/$$/ns/mnt
```

2. 列出 `/proc` 目录文件

```bash
$ ls /proc
1      1402  1572    23      31402    434     763    9539       locks
10     1407  1591    23306   31410    4346    764    9585       mdstat
1020   1411  16      24      318      44      767    961        meminfo
1033   1415  16276   250     32       45      768    9656       misc
1057   1420  163     251     325      4668    769    acpi       modules
……
# 很多，不一一列出
```

因为此时的 `/proc` 还是宿主机的，所以存在大量文件，必须手动重新挂载。

3. 手动重新挂载 `/proc` 目录

```bash
$ mount -t proc proc /proc
$ ls /proc
1          dma          key-users    net           sysrq-trigger
4          driver       kmsg         pagetypeinfo  sysvipc
acpi       execdomains  kpagecgroup  partitions    thread-self
……
# 少了很多，但还是不少，不一一列出
```

> **⚠️**：理论上该命令的执行不影响宿主机（或者我理解有误），但实际上还是影响了，我暂时不知道原因。

## 命令行演示

1. 制作两个 ISO 文件，之后用于挂载

```bash
$ mkdir iso && cd iso
$ mkdir -p iso1/subdir1 && mkdir -p iso2/subdir2
$ mkisofs -input-charset utf-8 -o ./1.iso ./iso1 && mkisofs -input-charset utf-8 -o ./2.iso ./iso2

$ ls
1.iso  2.iso  iso1/  iso2/
```

2. 在 `/mnt` 中创建两个挂载目录

```bash
$ mkdir /mnt/iso1 /mnt/iso2
```

3. 当前 Mount 命名空间

```bash
$ readlink /proc/$$/ns/mnt
mnt:[4026531840]
```

4. 将 `1.iso` 文件挂载到 `/mnt/iso1`

```bash
$ mount ./1.iso /mnt/iso1
mount: /mnt/iso1: WARNING: device write-protected, mounted read-only.

$ mount | grep 1.iso
/root/iso/1.iso on /mnt/iso1 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)

# 挂载成功
```

5. 加入新的 Mount 命名空间

```bash
$ unshare --mount /bin/bash

# 新的 Mount 命名空间
$ readlink /proc/$$/ns/mnt
mnt:[4026532440]

# 拷贝过来的挂载信息
$ mount | grep 1.iso
/root/iso/1.iso on /mnt/iso1 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)
```

6. 在新的 Mount 命名空间挂载 `2.iso` 文件

```bash
$ mount ./2.iso /mnt/iso2
mount: /mnt/iso1: WARNING: device write-protected, mounted read-only.

# 此时两个文件都已挂载
$ mount | grep iso
/root/iso/1.iso on /mnt/iso1 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)
/root/iso/2.iso on /mnt/iso2 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)
```

7. 卸载 `1.iso` 文件

```bash
$ umount /mnt/iso1
$ mount | grep iso
/root/iso/2.iso on /mnt/iso2 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)
```

8. 打开新的终端窗口，查看挂载信息

```bash
$ mount | grep iso
/root/iso/1.iso on /mnt/iso1 type iso9660 (ro,relatime,nojoliet,check=s,map=n,blocksize=2048)
```

可见原 Mount 命名空间的挂载信息没有改变。

### 章节导航

- 上一节：[PID 命名空间](PID%20命名空间.md)
- 下一节：[User 命名空间](User%20命名空间.md)
