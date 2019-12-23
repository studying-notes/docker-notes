# 操作 CPU 子系统（subsystem）

> 通过 subsystem 限制 cgroup 中与 CPU 相关的资源。

在上一篇笔记中创建 hierarchy 时，hierarchy 没有关联到任何的 subsystem，所以没办法通过那个 hierarchy 中的 cgroup 节点限制进程的资源占用。其实系统默认已经为每个 subsystem 创建了一个默认的 hierarchy，我们可以直接使用现成的。

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

设置 CPU 的分配比重，针对所有的 CPU（内核），默认值是 1024。假如系统中有两个 cgroup，分别是 A 和 B，A 的 shares 值是1024，B 的 shares 值是512，那么 A 将获得 1024/(1204+512)=66% 的 CPU 资源，而 B 将获得 33% 的 CPU 资源。cpu.shares 有两个特点：

- 如果 A 不忙，没有使用到 66% 的 CPU 时间，那么剩余的 CPU 时间将会被系统分配给 B，即 B 的 CPU 使用率可以超过 33%；

- 如果添加了一个新的 cgroup C，且它的 shares 值是1024，那么A的限额变成了1024/(1204+512+1024)=40%，B 的变成了 20%。

从上面两个特点可以看出：

- 在闲的时候，shares 基本上不起作用，只有在 CPU 忙的时候起作用，这是一个优点。
- 由于 shares 是一个绝对值，需要和其它 cgroup 的值进行比较才能得到自己的相对限额，而在一个部署很多容器的机器上，cgroup 的数量是变化的，所以这个限额也是变化的，自己设置了一个高的值，但别人可能设置了一个更高的值，所以这个功能没法精确的控制 CPU 占用率。

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

- 上一节：[操作控制组树](操作控制组树.md)
- 下一节：[操作进程子系统](操作进程子系统.md)
