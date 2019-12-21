# Cgroup 命名空间

> Cgroup 命名空间可虚拟化进程的 cgroup 的视图，每个 Cgroup 命名空间都有其自己的 cgroup 根目录集合。

**当前终端所属 Cgroup 命名空间视图**

```bash
$ cat /proc/self/cgroup
12:blkio:/user.slice
11:cpu,cpuacct:/user.slice
10:hugetlb:/
9:perf_event:/
8:net_cls,net_prio:/
7:rdma:/
6:freezer:/
5:cpuset:/
4:devices:/user.slice
3:pids:/user.slice/user-0.slice/session-246.scope
2:memory:/user.slice
1:name=systemd:/user.slice/user-0.slice/session-246.scope
0::/user.slice/user-0.slice/session-246.scope
```

## Go 语言实现 Cgroup 命名空间隔离

与之前不同的是，Go 标准库尚未引入这个系统调用，因此必须自行定义该参数。

```go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)


// 自行定义
const CLONE_NEWCGROUP = 0x02000000
// 参考：https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/sched.h

func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS | syscall.CLONE_NEWUSER | syscall.CLONE_NEWNET | CLONE_NEWCGROUP,
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

1. 宿主机所属的 Cgroup 命名空间

```bash
$ readlink /proc/self/ns/cgroup
cgroup:[4026531835]
```

2. 执行 `go run cgroup.go` 命令，进入另一个交互终端

```bash
$ go run cgroup.go

# 新的 Cgroup 命名空间
$ readlink /proc/self/ns/cgroup
cgroup:[4026532611]
```

3. 新 Cgroup 命名空间的 cgroup 视图

```bash
$ cat /proc/self/cgroup 
12:blkio:/
11:cpu,cpuacct:/
10:hugetlb:/
9:perf_event:/
8:net_cls,net_prio:/
7:rdma:/
6:freezer:/
5:cpuset:/
4:devices:/
3:pids:/
2:memory:/
1:name=systemd:/
0::/

# 对比之前的宿主机的 `cgroup` 视图，可以发现两者不一样
```

## 命令行创建新 Cgroup 命名空间示例

> 参考：http://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html

1. 以 freezer 为根，创建一个子 cgroup，然后放入一个进程

```bash
$ mkdir -p /sys/fs/cgroup/freezer/sub2

# 创建一个进程
$ sleep 10000 & 
[1]20544

# 将此进程放入该 cgroup
$ echo 20544  > /sys/fs/cgroup/freezer/sub2/cgroup.procs

$ cat /proc/20544/cgroup | grep freezer
6:freezer:/sub2
```

2. 再创建一个子 cgroup，然后放入终端进程

```bash
$ mkdir -p /sys/fs/cgroup/freezer/sub

# 放入终端进程
$ echo $$
24437

$ echo 24437  > /sys/fs/cgroup/freezer/sub/cgroup.procs

$ cat /proc/self/cgroup | grep freezer
6:freezer:/sub
```

3. 创建新的 Cgroup 和 Mount 命名空间

```bash
$ unshare -Cm bash

# 新 Cgroup 命名空间中该终端的 cgroup freezer 视图
# 与 `cat /proc/24437/cgroup | grep freezer` 等价
$ cat /proc/self/cgroup | grep freezer
6:freezer:/

# 新 Cgroup 命名空间中1号进程的 cgroup freezer 视图
$ cat /proc/1/cgroup | grep freezer
6:freezer:/..

# 新 Cgroup 命名空间中20544号进程的 cgroup freezer 视图
$ cat /proc/20544/cgroup | grep freezer
6:freezer:/../sub2

$ cat /proc/self/mountinfo | grep freezer
1333 1326 0:35 /.. /sys/fs/cgroup/freezer ...
```

从输出中我们可以看到新终端的 cgroup 成员关系是相对于  `freezer cgroup` 根目录（即新终端进程所在的 cgroup 目录，这里是 `sub`）定义的，不再是原来的目录 `freezer`。

但是最后一个命令输出了属于初始 Cgroup 命名空间的 `cgroup filesystem`，是错误的相对路径，为了解决这个问题，必须重新挂载 `freezer cgroup filesystem`。

4. 修复问题

```bash
$ mount --make-rslave /
$ umount /sys/fs/cgroup/freezer
$ mount -t cgroup -o freezer freezer /sys/fs/cgroup/freezer

$ cat /proc/self/mountinfo | grep freezer
1333 1326 0:35 / /sys/fs/cgroup/freezer ...
```

## Cgroup 命名空间的好处

- Cgroup 命名空间可以限制容器的 `cgroup filesytem` 视图，进而在容器中也可以安全的使用 `cgroup`；
- **让容器迁移更加容易**；在迁移时，`/proc/self/cgroup` 需要复制到目标机器，这要求容器的 `cgroup` 路径是唯一的，否则可能会与目标机器冲突。有了 Cgroup 命名空间，每个容器都有自己的 `cgroup filesystem` 视图，不用再担心这种冲突。

### 章节导航

- 上一节：[Network 命名空间](Network%20命名空间.md)
- 下一节：[自制一个简易容器](自制一个简易容器.md)
