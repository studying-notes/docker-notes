# ` docker update` 命令详解

> 动态更新一个或多个容器的配置

## 基本信息

### 用法

```
docker update [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 描述 |
| ------ | ------ |
| `--blkio-weight` | Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0) |
| `--cpu-period` | Limit CPU CFS (Completely Fair Scheduler) period |
| `--cpu-quota` | Limit CPU CFS (Completely Fair Scheduler) quota |
| `--cpu-rt-period` | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Limit the CPU real-time period in microseconds |
| `--cpu-rt-runtime` | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Limit the CPU real-time runtime in microseconds |
| `--cpu-shares , -c` | CPU shares (relative weight) |
| `--cpus` | [**API 1.29+**](https://docs.docker.com/engine/api/v1.29/) Number of CPUs |
| `--cpuset-cpus` | CPUs in which to allow execution (0-3, 0,1) |
| `--cpuset-mems` | MEMs in which to allow execution (0-3, 0,1) |
| `--kernel-memory` | Kernel memory limit |
| `--memory , -m` | Memory limit |
| `--memory-reservation` | Memory soft limit |
| `--memory-swap` | Swap limit equal to memory plus swap: ‘-1’ to enable unlimited swap |
| `--pids-limit` | [**API 1.40+**](https://docs.docker.com/engine/api/v1.40/) Tune container pids limit (set -1 for unlimited) |
| `--restart` | Restart policy to apply when a container exits |

## 扩展信息

- 同时更新多个容器可用空格分隔
- 不支持 Windows 容器

## 示例

### Update a container’s cpu-shares

```bash
$ docker update --cpu-shares 512 abebf7571666
```

### Update a container's memory

```bash
$ docker update --cpu-shares 512 -m 300M abebf7571666 hopeful_morse
```

### Update a container’s restart policy

Note that if the container is started with “--rm” flag, you cannot update the restart policy for it. The `AutoRemove` and `RestartPolicy` are mutually exclusive for the container.

```bash
$ docker update --restart=on-failure:3 abebf7571666 hopeful_morse
```

### Update a container’s kernel memory constraints

On kernel version older than 4.6, this option can be updated on a running container only if the container was started with `--kernel-memory`. If the container was started *without* `--kernel-memory` you need to stop the container before updating kernel memory.

```bash
$ docker run -dit --name test --kernel-memory 50M ubuntu bash
$ docker update --kernel-memory 80M test
```

```bash
$ docker run -dit --name test2 --memory 300M ubuntu bash
```

Update kernel memory of running container `test2` will fail. You need to stop the container before updating the `--kernel-memory` setting. The next time you start it, the container uses the new value.

Kernel version newer than (include) 4.6 does not have this limitation, you can use `--kernel-memory` the same way as other options.
