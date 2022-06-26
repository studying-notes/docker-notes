---
date: 2019-12-15T22:20:49+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "操作 CPU 子系统（subsystem）"  # 文章标题
url:  "posts/docker/cgroup/cpu"  # 设置网页链接，默认使用文件名
tags: [ "docker", "cgroup"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 文章在章节中的排序优先级，正序排序
chapter: false  # 将页面设置为章节

index: true  # 文章是否可以被索引
draft: false  # 草稿
---

> 通过 subsystem 限制 cgroup 中与 CPU 相关的资源。

## CPU 相关子系统

在 cgroup 里面，跟 CPU 相关的子系统有 `cpusets`、`cpuacct` 和 `cpu`。

- `cpuset` 限制 cgroup 中的进程只能在指定的 CPU 和 NUMA 内存节点上运行，基本参数如下：
    - `cpuset.cpus`：允许进程使用的 CPU 列表（示例：0~4，9）。
    - `cpuset.mems`：允许进程使用的 NUMA 内存节点列表（示例：0~1）。
- `cpuacct` 当前 cgroup 所使用 CPU 的统计信息。
    - `cpuacct.stat`：当前 cgroup 分别在用户态和内核态消耗的 CPU 时间，单位是 `USER_HZ`。`USER_HZ` 在 `x86` 上一般是100，即 `1 USER_HZ` 等于0.01秒。
    - `cpuacct.usage`：当前 cgroup 消耗的总 CPU 时间，单位是纳秒。
    - `cpuacct.usage_percpu`：当前 cgroup 在各个 CPU 上消耗的 CPU 时间，总和也就是 `cpuacct.usage` 的值。
- `cpu` 限制 cgroup 中进程的 CPU 相对占用率。

## CPU 子系统文件

1. 输出挂载信息

```bash
$ mount | grep cpu
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
```

2. 创建关联 cpu subsystem 的子 cgroup

```bash
$ cd /sys/fs/cgroup/cpu,cpuacct
$ mkdir cpu-demo && cd cpu-demo
```

3. 输出自带的初始文件

```bash
$ ls
cgroup.clone_children      cpuacct.usage_sys
cgroup.procs               cpuacct.usage_user
cpuacct.stat               cpu.cfs_period_us
cpuacct.usage              cpu.cfs_quota_us
cpuacct.usage_all          cpu.shares
cpuacct.usage_percpu       cpu.stat
cpuacct.usage_percpu_sys   notify_on_release
cpuacct.usage_percpu_user  tasks
```

**cpu.cfs_period_us** & **cpu.cfs_quota_us**

这两个文件配合起来可以设置 CPU 的占用上限。时间单位都是微秒。

- `cfs_period_us` 用来设置时间周期长度，取值范围为 1 毫秒到 1 秒。
- `cfs_quota_us` 用来设置当前 cgroup 在一个 `cfs_period_us` 内所能使用的 CPU 时间数，取值大于 1ms 即可。如果 `cfs_quota_us` 的值为 -1（默认值），表示不受 CPU 时间的限制。

**示例**

1. 限制只能使用 1 个 CPU（每 250ms 能使用 250ms 的 CPU 时间）

```bash
$ echo 250000 > cpu.cfs_quota_us
# quota = 250ms
$ echo 250000 > cpu.cfs_period_us
# period = 250ms
```

2. 限制使用 2 个 CPU（内核）（每 500ms 能使用 1000ms 的 CPU 时间）

```bash
$ echo 1000000 > cpu.cfs_quota_us
# quota = 1000ms
$ echo 500000 > cpu.cfs_period_us
# period = 500ms
```

3. 限制使用 1 个 CPU 的 20%（每 50ms 能使用 10ms 的 CPU 时间）

```bash
$ echo 10000 > cpu.cfs_quota_us
# quota = 10ms
$ echo 50000 > cpu.cfs_period_us
# period = 50ms
```

**cpu.shares**

用于设置 cgroup 的 CPU 的分配比重，针对所有的 CPU（内核），默认值是 1024。假如系统中有两个 cgroup，分别是 A 和 B，A 的 shares 值是1024，B 的 shares 值是512，那么理论上 A 将获得 1024/(1204+512)=66% 的 CPU 资源，而 B 将获得 33% 的 CPU 资源。实际上，`cpu.shares` 有几个特点：

- **该比例仅在运行 CPU 密集型进程时适用**。一个 cgroup 空闲时，其他 cgroup 可以消耗剩余 CPU 时间。就上面的例子来说，如果 A 不忙，没有使用到 66% 的 CPU 时间，那么剩余的 CPU 时间将会被系统分配给 B，即 B 的 CPU 使用率可以超过 33%。简而言之，在 CPU 空闲的时候，shares 基本上不起作用，只有在 CPU 忙的时候起作用。

- 在多核的机器上，CPU 时间份额分配在所有 CPU 内核上，即使 cgroup 被限制为少于 100% 的 CPU 时间，它也可以 100% 占用每个 CPU 核心。同样是上面的例子，但 CPU 有三个内核，那么这两个 cgroup 的 CPU 分配可能是：

```
PID    container	CPU	CPU share
100    B		0	100% of CPU0
101    A		1	100% of CPU1
102    A		2	100% of CPU2
```

- 假设在上面的例子中，另外加入了一个新的 cgroup C，且它的 shares 值是1024，那么 A 的限额变成了1024/(1204+512+1024)=40%，B 的变成了 20%。由于 shares 是一个绝对值，需要和其它 cgroup 的值进行比较才能得到自己的相对限额，而在一个部署很多容器的机器上，cgroup 的数量是变化的，所以这个限额也是变化的，自己设置了一个高的值，但别人可能设置了一个更高的值，所以这个功能没法精确的控制 CPU 占用率。

**cpu.stat**

记录 CPU 的统计信息，一共有三项：

1. **nr_periods**：表示过去了多少个 `cpu.cfs_period_us` 时间周期。
2.  **nr_throttled**：在过去的 `cpu.cfs_period_us` 时间周期中，有多少次是受到了限制，即 cgroup 中的进程多少次 CPU 占用达到了上限。
3. **throttled_time**: cgroup 中的进程被限制使用 CPU 持续了多长时间（纳秒）。

## 限制 CPU 实验

1. 限制使用 1 个 CPU 的 20%

```bash
$ echo 50000 > cpu.cfs_period_us
$ echo 10000 > cpu.cfs_quota_us

$ echo $$ > cgroup.procs
```

2. 启动一个死循环消耗 CPU

```bash
$ while :; do echo infinite > /dev/null; done
```

3. 另开一个窗口，运行 `top` 命令

```bash
$ top
……
# 可以看到上个终端 CPU 占用率差不多为 20%
```

### 章节导航

- 上一节：[操作进程子系统](操作进程子系统.md)
- 下一节：[操作内存子系统](操作内存子系统.md)
