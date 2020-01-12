# `docker pause` 命令详解

> 暂停一个或多个容器中的所有进程

## 基本信息

### 用法

```
docker pause CONTAINER [CONTAINER...]
```

## 扩展信息

- 在 Linux 上，这个功能通过 cgroups `freezer` 实现。过去在挂起进程时，会使用 `SIGSTOP` 信号，该信号可以被挂起的进程观察到。而通过  cgroups `freezer`，进程不知道也无法捕获它正在被挂起，以及最后恢复运行的中间过程进程完全无法感知。
- 在 Windows 上，只有 Hyper-V 容器可以被暂停。
- 更多信息可以参见 [cgroups freezer documentation](https://www.kernel.org/doc/Documentation/cgroup-v1/freezer-subsystem.txt)。

## 示例

```bash
$ docker pause container
```
