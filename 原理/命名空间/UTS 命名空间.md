# UTS 命名空间

> UTS 是 UNIX Timesharing System 的缩写，UTS 命名空间用于隔离主机名和域名两个系统标识。

## 显示当前主机名和域名

```bash
# 显示当前主机名
$ hostname
localhost

# 显示当前域名
$ domainname
localdomain
```

## Go 语言实现 UTS 命名空间隔离

```go
package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {

	// 指定被创建出来的新进程的初始命令
	cmd := exec.Command("sh")

	// 设置系统调用参数，Go 封装了系统调用 clone() 函数
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS,
	}

	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	// 执行代码，进入一个 sh 终端中
	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

## 实验过程

1. 执行 `go run uts.go` 命令，然后就进入了一个新的 *sh* 终端

```sh
$ go run uts.go

# 输出终端的进程号
$ echo $$
26468
```

2. 查看系统中进程之间的关系，这里只截取了关键部分

```sh
$ pstree -pl | grep grep
|-fish(26361)---go(26400)-+-uts(26463)-+-sh(26468)-+-grep(26618)
```

> **说明一下**：我用的终端是 *fish*，不是 *bash*，但这没有影响。

3. 验证这些进程是否在同一个 UTS 命名空间中

```sh
# 子进程所在 UTS 命名空间
$ readlink /proc/26468/ns/uts 
uts:[4026532440]

# 祖先进程所在 UTS 命名空间
$ readlink /proc/26463/ns/uts
uts:[4026531838]
$ readlink /proc/26400/ns/uts
uts:[4026531838]
$ readlink /proc/26361/ns/uts
uts:[4026531838]
```

可见唯有 *sh* 进程拥有独立的 UTS 命名空间，而它的祖先进程都同属一个 UTS 命名空间。

4. 验证 *sh* 子进程的 UTS 命名空间

```sh
$ bash
$ pstree -pl | grep grep
|-fish(26361)---go(26400)-+-uts(26463)-+-sh(26468)---bash(27247)-+-grep(27258)

$ readlink /proc/27247/ns/uts
uts:[4026532440]
```

由此可见，其创建的子进程和它位于同一个 UTS 命名空间中。

由于 UTS 命名空间对主机名进行了隔离，所以在这个空间中修改主机名不影响外部主机。

5. 修改主机名

```bash
$ hostname -b centos
$ hostname
centos
```

6. 然后启动另外一个终端，在宿主机上运行`hostname`命令

```bash
$ hostname
localhost
```

可以看到，外部主机名并没有被内部修改所影响，域名修改也是同理，由此可以了解 UTS 命名空间的作用。

### 章节导航

- 上一节：[命名空间概述](原理/命名空间/命名空间概述.md)
- 下一节：[IPC 命名空间](原理/命名空间/IPC 命名空间.md)