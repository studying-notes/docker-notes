# IPC 命名空间

> IPC 是 Inter-Process Communication 的缩写，中文名即进程间通信。进程是计算机系统分配资源的最小单位，每个进程都有自己的一部分独立的系统资源，彼此是隔离的。为了能使不同的进程互相访问资源并进行协调工作，才有了进程间通信。IPC 命名空间用于隔离不同命名空间中进程间的通信。

## Go 语言实现 IPC 命名空间隔离

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

	// 与上节代码相比仅增加了一个 CLONE_NEWIPC 参数
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC,
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

1. 列出现有的进程间通信

```bash
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

2. 在宿主机上创建一个进程间通信

```bash
$ ipcmk -Q
Message queue id: 0

# 复查，发现一个新的进程间通信列队
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0xfc0703a1 0          root       644        0            0
```

3. 执行 `go run ipc.go` 命令，进入另一个交互终端

```bash
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

在新创建的命名空间中，看不到宿主机中的进程间通信列队，可见 IPC 已被隔离。

## 命令行示例

Ubuntu 18.04.3 LTS 中自带两个与命名空间相关的命令：

```bash
$ nsenter --help
Run a program with namespaces of other processes.
# 运行一个程序，让它加入其他进程的命名空间，然后执行参数中指定的命令

$ unshare --help
Run a program with some namespaces unshared from the parent.
# 在若干个不与父进程共享的命名空间中运行程序，然后执行参数中指定的命令
```

因此能够方便地通过终端创建、操作命名空间。

**1号终端**

```bash
# 进程号
$ echo $$
26259

# 该终端所在命名空间
$ readlink /proc/$$/ns/uts /proc/$$/ns/ipc
uts:[4026531838]
ipc:[4026531839]

# 确认主机名
$ hostname
ubuntu

# 清除所有进程间通信
$ ipcrm -a

# 确认当前进程间通信
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

# 创建一个进程间通信列队
$ $ ipcmk -Q
Message queue id: 0

# 复查，发现一个新的进程间通信列队
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0xf124383f 32768      root       644        0            0
```

**2号终端**

```bash
# 该终端所在命名空间，与1号终端一致
$ readlink /proc/$$/ns/uts /proc/$$/ns/ipc
uts:[4026531838]
ipc:[4026531839]

$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0xf124383f 32768      root       644        0            0

# 理论上是该进程离开当前 UTS 和 IPC 命名空间
# 创建加入新的 UTS 和 IPC 命名空间，然后执行
# 参数，启动一个终端替换当前进程
$ unshare -iu /bin/bash
# 但我不太理解，下文有所说明

$ readlink /proc/$$/ns/uts /proc/$$/ns/ipc
uts:[4026532440]
ipc:[4026532441]

# 更改主机名，方便与1号终端区分
$ hostname -b centos && exec bash
$ hostname
centos

# 进程间关系
$ pstree -pl | grep grep
|-bash(26553)---bash(27001)-+-grep(27223)
# 这表明该命令其实是创建了子进程，将子进程放入了新的命名空间，
# 而自己没有离开当前命名空间（或者我理解有误，才入门水平）

# 确认当前进程间通信
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
# 已经被隔离

# 尝试进入1号终端的命名空间
$ nsenter -t 26259 -u -i /bin/bash
$ readlink /proc/$$/ns/uts /proc/$$/ns/ipc
uts:[4026531838]
ipc:[4026531839]

# 成功进入
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0xf124383f 32768      root       644        0            0

# 但是当我再次查看进程间关系，似乎是又创建了子进程
|-bash(26553)---bash(27001)---bash(27163)-+-grep(27212)
```

> **⚠️**：关于两个自带命令在实验过程中的不解，因为才疏学浅，不敢妄下定论，所以不在这误人子弟了。

### 章节导航

- 上一节：[UTS 命名空间](UTS%20命名空间.md)
- 下一节：[PID 命名空间](PID%20命名空间.md)
