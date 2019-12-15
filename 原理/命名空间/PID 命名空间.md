# PID 命名空间

> PID 是 Process Identifier 的缩写，PID 命名空间用于隔离进程号。同一个进程在不同的 PID 命名空间可以拥有不同的进程号，不同 PID 命名空间的进程号可以重复，互不影响。


## PID 命名空间嵌套

PID 命名空间可以嵌套，也就是说有父子关系，在当前命名空间里创建的所有新命名空间都是当前命名空间的子命名空间。**父命名空间里可以看到所有子孙后代命名空间里的进程信息，且可以发送信号给子命名空间的进程，但是父子空间中各自看到的进程号是不一样的**，且子命名空间里看不到祖先或者兄弟命名空间里的进程信息。

目前 PID 命名空间最多可以嵌套32层，由内核中的宏 `MAX_PID_NS_LEVEL` 来定义。

Linux 下的每个进程都有一个对应的 `/proc/pid` 目录，该目录包含了有关当前进程的信息。对一个 PID 命名空间而言，`/proc` 目录只包含当前命名空间和它所有子孙后代命名空间里的进程的信息。

在 Linux 系统中，进程号从1开始往后不断增加，进程退出后，可以回收再利用。进程号为1的进程是内核启动的第一个应用层进程，一般是 `init` 进程（现在采用 `systemd` 的系统第一个进程是 `systemd`）。当系统中一个进程的父进程退出时，内核会指定 `init` 进程成为这个进程的新父进程，而当 `init` 进程退出时，系统也将退出。

除非在 `init` 进程里指定了处理信号，否则内核会帮 `init` 进程屏蔽掉其他任何信号，这样可以防止其他进程不小心终止 `init` 进程导致系统崩溃。不过有了 PID 命名空间后，可以通过在父命名空间中发送 `SIGKILL` 或者 `SIGSTOP` 信号来终止子命名空间中的进程号为1的进程。

由于进程号为1的进程的特殊性，所以每个 PID 命名空间的第一个进程的编号都是1。当这个进程运行停止后，内核将会给这个命名空间里的所有其他进程发送 `SIGKILL` 信号，致使其他所有进程都停止，然后该命名空间会被销毁。

## Go 语言实现 PID 命名空间隔离

因为 Mount 命名空间的关系，此部分代码无法隔离 `/proc/pid` 目录，必须配合 Mount 命名空间才可以实现，或者在实验中手动重新挂载该目录。

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
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID,
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

1. 执行 `go run pid.go` 命令

```bash
$ go run pid.go

# 在出现的交互终端中查看进程树
$ pstree -pl | grep grep
|-go(31988)-+-pid(32046)-+-sh(32051)-+-grep(32111)

# 输出此终端进程号
$ echo $$
1
```

可以看到，`sh(32051)` 在新的命名空间中被映射为1。但这里不能用 `ps` 和 `top` 命令查看，因为这两个命令会使用 `/proc` 目录中的内容。

2. 手动重新挂载 `/proc` 目录

```bash
$ mount -t proc proc /proc
$ ps
  PID TTY          TIME CMD
    1 pts/3    00:00:00 sh
    5 pts/3    00:00:00 ps

# 再次检查进程树，发现已经无法看到祖先进程信息了
$  pstree -pl
sh(1)───pstree(9)
```

> **⚠️**：因为 Mount 命名空间未隔离，此命令会影响宿主机，但问题不大，宿主机再次手动挂载即可。

## 空间嵌套示例

方便起见，改用终端自带命令演示。

**1号终端**

1. 记下最外层的 PID 命名空间

```bash
$ readlink /proc/$$/ns/pid
pid:[4026531836]
```

2. 创建新的 PID 命名空间

```bash
$ unshare --uts --pid --mount --fork --mount-proc /bin/bash
```

- `--mount-proc` 参数是让 `unshare` 自动重新挂载 `mount/proc` 目录。
- `--fork` 是为了让进程 `fork` 一个新的进程出来，然后再用终端替换掉新的进程。这是 PID 命名空间的限制，进程所属的 PID 命名空间在它创建的时候就确定了，不能更改，所以调用 `unshare` 和 `nsenter` 后，原来的进程还是属于原来的命名空间，而新 `fork` 出来的进程才属于新的命名空间，这也算解释了我之前的疑问。

3. 修改主机名，方便区别

```bash
$ hostname -b h1 && exec bash
$ readlink /proc/$$/ns/pid
pid:[4026532442]
```

4. 创建多个嵌套 PID 命名空间
```bash
$ unshare --uts --pid --mount --fork --mount-proc /bin/bash
$ hostname -b h2 && exec bash
$ readlink /proc/$$/ns/pid
pid:[4026532446]

$ unshare --uts --pid --mount --fork --mount-proc /bin/bash
$ hostname -b h3 && exec bash
$ readlink /proc/$$/ns/pid
pid:[4026532449]
```

5. 最后在 `h3` 中查看进程树

```bash
$ pstree -pl
bash(1)───pstree(22)
```

至此，在1号终端，我们建立了三层 PID 命名空间嵌套：`h1 -> h2 -> h3`。

**2号终端**

1. 在最外层的命名空间查看1号终端的进程树
```bash
$ pstree -pl | grep bash | grep unshare
|-bash(2730)---unshare(2753)---bash(2754)---unshare(2846)---bash(2847)---unshare(2878)---bash(2879)
```

2. 确认三个进程的 PID 命名空间

```bash
$ readlink /proc/2754/ns/pid
pid:[4026532442]

$ readlink /proc/2847/ns/pid
pid:[4026532446]

$ readlink /proc/2879/ns/pid
pid:[4026532449]
```

3. 进程号在各个命名空间中的映射关系

```bash
$ grep NSpid /proc/2879/status
NSpid:  2879    45      23      1
```

4. 创建一个新终端进程进入 `h1` 的命名空间

```bash
$ nsenter --uts --mount --pid -t 2754 /bin/bash
```

5. 进程树

```bash
$ pstree -pl
bash(1)───unshare(22)───bash(23)───unshare(44)───bash(45)

$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:28 pts/0    00:00:00 bash
root        22     1  0 08:31 pts/0    00:00:00 unshare --uts --pid --mount --for
root        23    22  0 08:31 pts/0    00:00:00 bash
root        44    23  0 08:32 pts/0    00:00:00 unshare --uts --pid --mount --for
root        45    44  0 08:32 pts/0    00:00:00 bash
root        68     0  0 10:00 pts/1    00:00:00 /bin/bash
root        83    68  0 11:02 pts/1    00:00:00 ps -ef
```

6. 尝试发送信号给 `h3` 子命名空间

```bash
$ kill 45
```

然后切换到1号终端，可以发现已经自动退出 `h3`，回到了 `h2`。

## 总结

- 当一个进程的父进程被 `kill` 掉后，该进程将会被当前命名空间中 `pid` 为1的进程接管，而不是被最外层的系统级别的 `init` 进程接管；
- 当 `pid` 为1的进程停止运行后，内核将会给这个命名空间及其子孙命名空间里的所有其他进程发送 `SIGKILL` 信号，致使其他所有进程都停止，于是当前命名空间及其子孙后代的命名空间都被销毁掉。

### 章节导航

- 上一节：[IPC 命名空间](IPC%20命名空间.md)
- 下一节：[Mount 命名空间](Mount%20命名空间.md)
