# Docker 中的控制组

Docker 是通过 Cgroups 实现容器资源限制和监控的。

## Docker 配置示例

### 通过 `docker run -m` 设置内存限制

```bash
$ docker run -d -m 128m ubuntu
902367fe722d2f0c73d46cb08bc1890c5657a6913e50741669297665669b64f3

# WARNING: Your kernel does not support swap limit capabilities or the cgroup is not mounted. Memory limited without swap.
```

然后我遇到了警告，为了让实验成功，必须解决这个问题。

1. 修改文件 `/etc/default/grub`，增加

```
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```

2. 更新该文件，然后重启

```bash
$ update-grub
$ shutdown -r now
```

> **⚠️**：亲身经历，千万别更改该文件原来的内容！

3. 恢复之前的实验

```bash
$ docker run -d -m 128m centos
68e330f6ba6194f5c1acf6d613ae3fbffcfacf447920744705bfa8227facdc79

$ cd /sys/fs/cgroup/memory/docker/68e330f6ba6194f5c1acf6d613ae3fbffcfacf447920744705bfa8227facdc79

$ ls
cgroup.clone_children               memory.memsw.failcnt
cgroup.event_control                memory.memsw.limit_in_bytes
cgroup.procs                           memory.memsw.max_usage_in_bytes
memory.failcnt                                  memory.memsw.usage_in_bytes
memory.force_empty                         memory.move_charge_at_immigrate
memory.kmem.failcnt                         memory.numa_stat
memory.kmem.limit_in_bytes               memory.oom_control
memory.kmem.max_usage_in_bytes      memory.pressure_level
memory.kmem.slabinfo                        memory.soft_limit_in_bytes
memory.kmem.tcp.failcnt                     memory.stat
memory.kmem.tcp.limit_in_bytes           memory.swappiness
memory.kmem.tcp.max_usage_in_bytes  memory.usage_in_bytes
memory.kmem.tcp.usage_in_bytes         memory.use_hierarchy
memory.kmem.usage_in_bytes              notify_on_release
memory.limit_in_bytes                          tasks
memory.max_usage_in_bytes

# 内存限制
$ cat memory.limit_in_bytes
134217728

# 已用内存
$ cat memory.usage_in_bytes
6332416 
```

可以看到，Docker 通过为每个容器创建 cgroup，并通过 cgroup 去配置资源限制和资源监控。
