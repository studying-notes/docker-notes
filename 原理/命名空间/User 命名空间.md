# User 命名空间

> User 命名空间用于隔离用户和用户组，一个进程的 User ID 和 Group ID 在 User 命名空间内外可以是不同的。一个进程可以在 User 命名空间里拥有 root 权限，但是在 User 命名空间外面可以没有 root 权限。

## Go 语言实现 User 命名空间隔离

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
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS | syscall.CLONE_NEWUSER,
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

1. 显示宿主机当前用户和用户组

```bash
$ id
uid=0(root) gid=0(root) groups=0(root)
```

2. 执行 `go run user.go` 命令

```bash
$ go run user.go
$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)

# 用户和用户组发生了变化
```

3. 作为对比，执行 `go run uts.go` 命令

```bash
$ go run uts.go
$ id
uid=0(root) gid=0(root) groups=0(root)

# 结论不言而喻
```

## User 命名空间的特殊性

除了 User 命名空间，创建其他类型的命名空间都需要 CAP_SYS_ADMIN 的 capability，即允许执行系统管理任务的能力。

> **扩展**：从 Linux Kernel 2.2 开始，与 root 用户关联的特权被划分为不同的单元，称为 **capabilites**，capabilites 作为线程的属性存在，每个单元可以独立启用和禁用。如此一来，权限检查的过程就变成了：在执行特权操作时，如果进程的有效身份不是 root，就去检查是否具有该特权操作所对应的 capabilites，并以此决定是否可以进行该特权操作。

但是一些特权操作不跟任何 User 命名空间关联，比如修改系统时间（CAP_SYS_MODULE）、创建设备（CAP_MKNOD），这些只能由系统启动后的默认 User 命名空间里有相应权限的进程来操作。

**命令行演示**：非 root 用户创建 User 命名空间

```bash
$ id
uid=1000(rustle) gid=1000(rustle) groups=1000(rustle)

$ readlink /proc/$$/ns/user
user:[4026531837]

$ unshare -- User -r /bin/bash
$ readlink /proc/$$/ns/user
user:[4026532440]

$ id
uid=0(root) gid=0(root) groups=0(root)
```

## 和其他类型命名空间的关系

Linux 下的每个命名空间，都有一个 User 命名空间和它关联，这个 User 命名空间就是创建相应命名空间时进程所属的 User 命名空间，相当于每个命名空间都有一个所有者（ User 命名空间），这样保证对任何命名空间的操作都受到 User 命名空间权限的控制。

## 和 Mount 命名空间的关系

当和 Mount 命名空间一起用时，不能挂载基于块设备的文件系统，但是可以挂载以下这些文件系统：

* /proc (since Linux 3.8)
* /sys (since Linux 3.8)
* devpts (since Linux 3.9)
* tmpfs(5) (since Linux 3.9)
* ramfs (since Linux 3.9)
* mqueue (since Linux 3.9)
* bpf (since Linux 4.4)

> 参见：http://man7.org/linux/man-pages/man7/user_namespaces.7.html

**命令行演示**

1. 创建新的 User 和 Mount 命名空间

```bash
$ unshare --user -r --mount bash

# 创建挂载测试目录
$ mkdir mnt
```

2. 尝试将块设备挂载到其他目录

```bash
# 查找挂载到根目录的块设备
$ mount | grep " / "
/dev/sda2 on / type ext4 (rw,relatime,errors=remount-ro)

# 确认是块设备
$ file /dev/sda2
/dev/sda2: block special (8/2)

$ mount /dev/sda2 mnt
mount: /root/mnt: /dev/sda2 already mounted on /.
```

3. 尝试挂载 `/proc` 目录

```bash
$ mount -t proc none mnt
mount: /root/mnt: permission denied.
```

> 由于当前 PID 命名空间不属于当前 User 命名空间，所以挂载 `/proc` 目录失败。

4. 创建新的 PID 命名空间，再次尝试挂载

```bash
$ unshare --pid --fork bash
$ mount -t proc none mnt
$ mount | grep mnt | grep proc
none on /root/mnt type proc (rw,relatime)

# 挂载成功！
```

5. 尝试挂载 `devpts` 目录

```bash
$ mount -t devpts devpts mnt
$ mount | grep mnt | grep devpts
devpts on /root/mnt type devpts (rw,relatime,mode=600,ptmxmode=000)

# 挂载成功！
```

6. 尝试挂载 `/sys` 目录

```bash
$ mount -t sysfs sysfs mnt
mount: /root/mnt: permission denied.
# 无法挂载，原因未知

# 假挂载
$ mount -vft sysfs sysfs mnt
mount: sysfs mounted on /root/mnt.
```

7. 尝试挂载 `tmpfs` 目录

```bash
$ mount -t tmpfs tmpfs mnt
$ mount | grep mnt | grep tmpfs
tmpfs on /root/mnt type tmpfs (rw,relatime)

# 挂载成功！
```

> `ramfs` 和 `tmpfs` 类似，都是内存文件系统，不再演示。

## 再次理解两个函数

Linux 中进程号与线程号都是以 `PID` 方式显示，所以 `unshare()` 创建的是新的线程，让新的线程加入了新的命名空间，除了这个线程，该进程的其他线程都没有加入这个新的命名空间，而 `clone()` 创建的是新的子进程，让新的子进程加入了新的命名空间。

### 章节导航

- 上一节：[Mount 命名空间](Mount%20命名空间.md)
- 下一节：[Network 命名空间](Network%20命名空间.md)
