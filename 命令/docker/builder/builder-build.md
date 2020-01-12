# `docker builder build` 命令

> 从 Dockerfile 构建镜像

## 基本信息

### 描述

估计是 `docker build` 命令的未来形式，用法与命令都是相同的，忘了之前在哪看到将来可能会取消一级操作命令。

仅 [**API 1.31**](https://docs.docker.com/engine/api/v1.31/) 及以上版本支持这个命令。

### 用法

```
docker builder build [OPTIONS] PATH | URL | -
```

### 选项

| 名称 | 默认 | 描述 |
| ------------------------- | ------- | ------------------------------------------------------------ |
| `--add-host` | | Add a custom host-to-IP mapping (host:ip) |
| `--build-arg` | | Set build-time variables |
| `--cache-from` | | Images to consider as cache sources |
| `--cgroup-parent` | | Optional parent cgroup for the container |
| `--compress` | | Compress the build context using gzip |
| `--cpu-period` | | Limit the CPU CFS (Completely Fair Scheduler) period |
| `--cpu-quota` | | Limit the CPU CFS (Completely Fair Scheduler) quota |
| `--cpu-shares , -c` | | CPU shares (relative weight) |
| `--cpuset-cpus` | | CPUs in which to allow execution (0-3, 0,1) |
| `--cpuset-mems` | | MEMs in which to allow execution (0-3, 0,1) |
| `--disable-content-trust` | `true` | Skip image verification |
| `--file , -f` | | Name of the Dockerfile (Default is ‘PATH/Dockerfile’) |
| `--force-rm` | | Always remove intermediate containers |
| `--iidfile` | | Write the image ID to the file |
| `--isolation` | | Container isolation technology |
| `--label` | | Set metadata for an image |
| `--memory , -m` | | Memory limit |
| `--memory-swap` | | Swap limit equal to memory plus swap: ‘-1’ to enable unlimited swap |
| `--network` | | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Set the networking mode for the RUN instructions during build |
| `--no-cache` | | Do not use cache when building the image |
| `--output , -o` | | [**API 1.40+**](https://docs.docker.com/engine/api/v1.40/) Output destination (format: type=local,dest=path) |
| `--platform` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.32+**](https://docs.docker.com/engine/api/v1.32/) Set platform if server is multi-platform capable |
| `--progress` | `auto` | Set type of progress output (auto, plain, tty). Use plain to show container output |
| `--pull` | | Always attempt to pull a newer version of the image |
| `--quiet , -q` | | Suppress the build output and print image ID on success |
| `--rm` | `true` | Remove intermediate containers after a successful build |
| `--secret` | | [**API 1.39+**](https://docs.docker.com/engine/api/v1.39/) Secret file to expose to the build (only if BuildKit enabled): id=mysecret,src=/local/secret |
| `--security-opt` | | Security options, only Windows |
| `--shm-size` | | Size of /dev/shm |
| `--squash` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Squash newly built layers into a single new layer |
| `--ssh` | | [**API 1.39+**](https://docs.docker.com/engine/api/v1.39/) SSH agent socket or keys to expose to the build (only if BuildKit enabled) (format: default\|[=\|[,]]) |
| `--stream` | | [**experimental (daemon)**](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)[**API 1.31+**](https://docs.docker.com/engine/api/v1.31/) Stream attaches to server to negotiate build context |
| `--tag , -t` | | Name and optionally a tag in the ‘name:tag’ format |
| `--target` | | Set the target build stage to build. |
| `--ulimit` | | Ulimit options |
