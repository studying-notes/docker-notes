# `dockerd` 命令详解

> 守护进程，https://docs.docker.com/engine/reference/commandline/dockerd/

- [`dockerd` 命令详解](#dockerd-命令详解)
	- [基本信息](#基本信息)
		- [用法](#用法)
		- [选项](#选项)
		- [描述](#描述)
	- [示例](#示例)
		- [Daemon Socket Option](#daemon-socket-option)
			- [BIND DOCKER TO ANOTHER HOST/PORT OR A UNIX SOCKET](#bind-docker-to-another-hostport-or-a-unix-socket)
		- [Daemon storage-driver](#daemon-storage-driver)
		- [Options per storage driver](#options-per-storage-driver)
			- [OVERLAY2 OPTIONS](#overlay2-options)
		- [Docker runtime execution options](#docker-runtime-execution-options)
			- [OPTIONS FOR THE RUNTIME](#options-for-the-runtime)
		- [Daemon DNS options](#daemon-dns-options)
		- [Allow push of nondistributable artifacts](#allow-push-of-nondistributable-artifacts)
		- [Insecure registries](#insecure-registries)
		- [Running a Docker daemon behind an HTTPS_PROXY](#running-a-docker-daemon-behind-an-https_proxy)
		- [Default `ulimit` settings](#default-ulimit-settings)
		- [Node discovery](#node-discovery)
		- [Access authorization](#access-authorization)
		- [Miscellaneous options](#miscellaneous-options)
			- [DEFAULT CGROUP PARENT](#default-cgroup-parent)
			- [DAEMON METRICS](#daemon-metrics)
			- [NODE GENERIC RESOURCES](#node-generic-resources)
		- [Daemon configuration file](#daemon-configuration-file)
			- [On Linux](#on-linux)
			- [On Windows](#on-windows)
			- [FEATURE OPTIONS](#feature-options)
			- [CONFIGURATION RELOAD BEHAVIOR](#configuration-reload-behavior)
		- [Run multiple daemons](#run-multiple-daemons)

## 基本信息

### 用法

```
dockerd COMMAND
```

### 选项

参数选项因为太多，另写成了单独的笔记，见[《`dockerd` 参数选项》](dockerd-opts.md)。

### 描述

- `dockerd` 是用于管理容器的常驻进程。Docker 分离了守护进程和客户端两个二进制程序。`dockerd -D` 可以打开调试模式，或者添加 `"debug": true` 到 `daemon.json` 文件。
- 启用 Docker 服务器端的实验性功能 `dockerd --experimental`，或者添加 `"experimental": true` 到 `daemon.json` 文件。

## 示例

### Daemon Socket Option

Docker daemon 可以通过三种类型的 Socket 监听  [Docker Engine API](https://docs.docker.com/engine/reference/api/) 请求：

1. `unix` 默认域 Socket 类型（IPC Socket），仅允许 root 用户连接，位置在 `/var/run/docker.sock`，必须拥有 root 权限才能连接，或者将当前用户加入 `docker` 用户组。默认参数 `unix:///var/run/docker.sock`。

2. `tcp` 可以用于远程访问 Docker daemon，默认设置仅提供了未加密和无身份认证的直接访问，为了更加安全，可以使用内置的 HTTPS 加密 Socket，或者加入安全的网络代理。默认情况下监听 `2375` 端口，既可以配置 `-H tcp://0.0.0.0:2375` 允许全部网络接口的请求，也可以设置 `-H tcp://192.168.59.103:2375` 仅允许来自指定的网络接口的请求。一般约定，`2375` 端口用于非加密，`2376` 用于加密通信。

3. `fd` 在基于 Systemd 的系统上，可以设置 `dockerd -H fd://` 指定 [Systemd socket activation](http://0pointer.de/blog/projects/socket-activation.html) 进行通信，也可以指定一个专门的 Socket `dockerd -H fd://3`，但若是该文件不存在就会导致 Docker 退出。

通过设置多个 `-H` 参数，Docker daemon 可以同时监听多个类型的 Socket：

```bash
# listen using the default unix socket, and on 2 specific IP addresses on this host.

$ dockerd -H unix:///var/run/docker.sock -H tcp://192.168.59.106 -H tcp://10.10.10.2
```

**永久绑定到 TCP 协议**

```bash
# 查询配置文件的存放位置
$ systemctl show --property=FragmentPath docker
FragmentPath=/lib/systemd/system/docker.service

# 在原来基础上添加参数
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0 -H fd:// --containerd=/run/containerd/containerd.sock

# 重启相关进程
$ systemctl daemon-reload
$ systemctl restart docker
```

Docker client 可以用 `DOCKER_HOST` 环境变量来为 client 设置 -H 参数，或者只是通过命令行一次性设置：

```bash
$ docker -H tcp://0.0.0.0:2375 ps
```

```bash
$ export DOCKER_HOST="tcp://0.0.0.0:2375"
$ docker ps
```

将 `DOCKER_TLS_VERIFY` 环境变量设置为任何非空值都等于设置了 `--tlsverify` 参数。

```bash
$ docker --tlsverify ps
# or
$ export DOCKER_TLS_VERIFY=1
$ docker ps
```

Docker client 支持 `HTTP_PROXY`, `HTTPS_PROXY` 和 `NO_PROXY` 环境变量，`HTTPS_PROXY` 优先于 `HTTP_PROXY`。

从 Docker 18.09 开始，Docker client 支持通过 SSH 连接远程 daemon：

```bash
$ docker -H ssh://me@example.com:22 ps
$ docker -H ssh://me@example.com ps
$ docker -H ssh://example.com ps
```

目前只支持通过公钥认证，不支持密码认证。

#### BIND DOCKER TO ANOTHER HOST/PORT OR A UNIX SOCKET

> 警告：将默认的 docker deamon 绑定到 TCP 端口或者 Unix docker 用户组将使非 root 用户获得 root 访问权限，从而增加安全风险，所以官方不推荐。

```
tcp://[host]:[port][path]
```

- `tcp://` -> TCP connection to `127.0.0.1` on either port `2376` when TLS encryption is on, or port `2375` when communication is in plain text.
- `tcp://host:2375` -> TCP connection on host:2375
- `tcp://host:2375/path` -> TCP connection on host:2375 and prepend path to all requests
- `unix://path/to/socket` -> Unix socket located at `path/to/socket`

`-H`, when empty, will default to the same value as when no `-H` was passed in.
`-H` also accepts short form for TCP bindings: `host:` or `host:port` or `:port`

```bash
$ dockerd -H 0.0.0.0:5555 &
```

```bash
$ docker -H :5555 pull ubuntu
```

```bash
# Run docker in daemon mode
$ <path to>/dockerd -H tcp://127.0.0.1:2375 -H unix:///var/run/docker.sock &
# Download an ubuntu image, use default Unix socket
$ docker pull ubuntu
# OR use the TCP port
$ docker -H tcp://127.0.0.1:2375 pull ubuntu
```

### Daemon storage-driver

在 Linux 上，Docker daemon 支持多种不同的 Image Layer 存储驱动：`aufs`, `devicemapper`, `btrfs`, `zfs`, `overlay`, `overlay2`。必须注意到存储驱动与其服务的文件系统之间的差别。

- `aufs` 最古老的驱动，但是该文件系统未并入主线，依赖内核补丁，已知有可能导致严重的内核崩溃。但是 aufs 允许容器分享程序和共享库内存，所以当运行上千个容器时或许很有用。
- `devicemapper` 不能和设备共享程序内存。
- `btrfs` 执行 `docker build` 很快，但是不能和设备共享程序内存，指定参数 `dockerd -s btrfs -g /mnt/btrfs_partition`。
- `zfs` 比 btrfs 稳定，指定参数 `dockerd -s zfs`。
- `overlay` 与之对应的 [OverlayFS](../../原理/联合文件系统/OverlayFS.md) 已并入主内核中。overlay 驱动支持页缓存共享，这意味着多个容器访问相同文件可以共享单个页缓存条目，这使得 overlay 与 aufs 在内存上一样高效。但是，随着镜像数量的增加，可能导致索引节点消耗过多以及与 RPMs 不兼容。所以不推荐用于生产环境，指定参数 `dockerd -s overlay`。
- `overlay2` 随着 [OverlayFS](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt) 引入[新特性](https://lkml.org/lkml/2015/2/11/106)，Docker 更新了 overlay 存储驱动，解决了索引节点的大量消耗问题，是现在的默认及推荐存储驱动，指定参数 `dockerd -s overlay2`。

在 Windows 上，Docker daemon 根据不同的 Image 平台支持唯一的 Image Layer 存储驱动：

- `windowsfilter` for Windows images
- `lcow` for Linux containers on Windows

### Options per storage driver

特定的存储驱动可以指定不同的 `--storage-opt` 选项参数进行自定义配置。
- `devicemapper` 前缀 `dm`
- `zfs` 前缀 `zfs`
- `btrfs` 前缀 `btrfs`
- `lcow` 前缀 `lcow`
- `overlay2` 前缀 `overlay2`

由于可以预见 overlay2 必将取代其他存储驱动，对于其他存储驱动这里不再展开。

#### OVERLAY2 OPTIONS

- overlay2.override_kernel_check 禁用内核版本检查。Linux Kernel 4.0.0 才增加了对多个 `lowerdir` 的支持，但是一些老版本可能通过补丁的方式支持了新特性，所以在确定内核支持的情况下，可以禁用版本检查。
- overlay2.size 设置容器的默认最大值，只有当其后端文件系统是 `xfs` 且以 `pquata` 参数挂载时，`size` 参数才可用。

```bash
$ dockerd -s overlay2 --storage-opt overlay2.size=1G
```

### Docker runtime execution options

Docker daemon 依赖于兼容 [OCI](https://github.com/opencontainers/runtime-spec) 标准的 runtime 实现作为 Linux 内核 `namespaces`、`cgroups` 和 `SELinux` 相关系统调用的接口，实际上 Docker 在 runtime 之上又进行了封装，即 `containerd` daemon。

默认情况下，Docker daemon 自动启动 `containerd`，也可以通过指定相关参数手动启动：

```bash
$ dockerd --containerd /var/run/dev/docker-containerd.sock
```

Runtimes 可以通过配置文件或者在命令行指定 `--add-runtime` 参数注册，可以设置多个 runtime。

```json
{
	"default-runtime": "runc",
	"runtimes": {
		"runc": {
			"path": "runc"
		},
		"custom": {
			"path": "/usr/local/bin/my-runc-replacement",
			"runtimeArgs": [
				"--debug"
			]
		}
	}
}
```

命令行方式不支持设置 `runtimeArgs`：

```bash
$ dockerd --add-runtime runc=runc --add-runtime custom=/usr/local/bin/my-runc-replacement
```

#### OPTIONS FOR THE RUNTIME

可以通过 `--exec-opt` 参数配置 runtime，全部参数值必须以 `native` 为前缀。

- `native.cgroupdriver` 指定容器控制组的管理驱动，只能是 `cgroupfs` or `systemd`，默认 ` cgroupfs`。

```bash
$ dockerd --exec-opt native.cgroupdriver=systemd
```

对全部用该 daemon 启动的容器有效。

### Daemon DNS options

为全部容器设置 DNS 服务器：

```bash
$ dockerd --dns 8.8.8.8
```

为全部容器设置 DNS 搜索域：

```bash
$ dockerd --dns-search example.com
```

### Allow push of nondistributable artifacts

一些镜像包含许可证限制分发的组件，当这些镜像被上传到 Registry 时不包含被限制的组件。

为了在特定的 Registry 上允许上传，可以用 `--allow-nondistributable-artifacts` 选项指定，支持两种形式：

- `--allow-nondistributable-artifacts myregistry:5000`
- `--allow-nondistributable-artifacts 10.1.0.0/16`

该参数可以同时指定多个 Registry。

### Insecure registries

Docker 允许私人 Registry 通过 HTTP，也可以通过 HTTPS。以下的示例中，用 `myregistry:5000` 作为占位符，表示私人 Registry。

一个采用了 TLS 的 Registry，会将 CA 凭证的副本存放在 Docker 主机的
`/etc/docker/certs.d/myregistry:5000/ca.crt` 中。一个不安全的 Registry 可能是未采用 TLS 技术，也可能是其 CA 凭证没有放置在 `/etc/docker/certs.d/myregistry:5000/` 下，或认证失败。

默认情况下，Docker 假定所有的 Registry 都是安全的（HTTPS 协议），因此不允许访问一个不安全的 Registry（HTTP 协议）。为了允许不安全的 Registry，必须指定 `--insecure-registry` 参数，有两种形式：

- `--insecure-registry myregistry:5000`
- `--insecure-registry 10.1.0.0/16`

### Running a Docker daemon behind an HTTPS_PROXY

当在局域网内使用 HTTPS 代理时，Docker Hub 凭证会被代理凭证替代，以下这些必须添加到 Docker 主机的配置中：

1. Install the `ca-certificates` package for your distribution
2. Ask your network admin for the proxy’s CA certificate and append them to `/etc/pki/tls/certs/ca-bundle.crt`
3. Then start your Docker daemon with `HTTPS_PROXY=http://username:password@proxy:port/ dockerd`. The `username:` and `password@` are optional - and are only needed if your proxy is set up to require authentication.

以上只是对 Docker daemon 的请求有效，对于 `docker build` 和运行容器还需要额外的配置。

### Default `ulimit` settings

`--default-ulimit` 允许为全部容器设置默认 `ulimit`。这和 `docker run` 的 `--ulimit` 参数是相同的。默认 `ulimit` 继承设置。即 `docker run` 未设置 `--ulimit` 时，从 Docker daemon 获取参数值。

小心地使用 `ulimit` 参数设置 `nproc`，因为 `nproc` 是为 Linux 设计用于设置用户而不是容器可用的最大进程数。

### Node discovery

Daemon 可以通过指定 `--cluster-advertise` 参数以 `host:port` or `interface:port` 的格式广播自身到集群之中，然后远程主机就可以通过这个值访问该 deamon。

Daemon 使用 [libkv](https://github.com/docker/libkv/) 在集群中广播节点。一些键值数据库支持双向 TLS，可以通过 `--cluster-store-opt` 参数指定 PEM 编码文件的路径进行配置，比如：

```bash
$ dockerd \
    --cluster-advertise 192.168.1.2:2376 \
    --cluster-store etcd://192.168.1.2:2379 \
    --cluster-store-opt kv.cacertfile=/path/to/ca.pem \
    --cluster-store-opt kv.certfile=/path/to/cert.pem \
    --cluster-store-opt kv.keyfile=/path/to/key.pem
```

当前支持的集群存储选项为：

| 选项 | 描述 |
| :----------- | :----------- |
| `discovery.heartbeat` | Specifies the heartbeat timer in seconds which is used by the daemon as a `keepalive` mechanism to make sure discovery module treats the node as alive in the cluster. If not configured, the default value is 20 seconds. |
| `discovery.ttl` | Specifies the TTL (time-to-live) in seconds which is used by the discovery module to timeout a node if a valid heartbeat is not received within the configured ttl value. If not configured, the default value is 60 seconds. |
| `kv.cacertfile` | Specifies the path to a local file with PEM encoded CA certificates to trust. |
| `kv.certfile` | Specifies the path to a local file with a PEM encoded certificate. This certificate is used as the client cert for communication with the Key/Value store. |
| `kv.keyfile` | Specifies the path to a local file with a PEM encoded private key. This private key is used as the client key for communication with the Key/Value store. |
| `kv.path` | Specifies the path in the Key/Value store. If not configured, the default value is ‘docker/nodes’. |

### Access authorization

Docker 的访问授权可以通过购买或自行构建的授权插件进行扩展。启动 Docker `daemon` 时，指定多个 `--authorization-plugin=PLUGIN_ID` 参数可以安装一个或多个认证插件。

```bash
$ dockerd --authorization-plugin=plugin1 --authorization-plugin=plugin2,...
```

`PLUGIN_ID` 可以是插件名，也可以是路径。

### Miscellaneous options

- IP 伪装通过 NAT 允许没有公共 IP 的容器与网络上的其他机器通信，但这可能会干扰某些网络拓扑，可以通过` --ip-masq=false` 禁用。

- Docker 支持 Docker 数据目录（`/var/lib/docker`）和临时目录（`/var/lib/docker/tmp`）的软连接。可以通过以下方式进行设置：

```
DOCKER_TMPDIR=/mnt/disk2/tmp /usr/local/bin/dockerd -D -g /var/lib/docker -H unix:// > /var/lib/docker-machine/docker.log 2>&1
# or
export DOCKER_TMPDIR=/mnt/disk2/tmp
/usr/local/bin/dockerd -D -g /var/lib/docker -H unix:// > /var/lib/docker-machine/docker.log 2>&1
```

#### DEFAULT CGROUP PARENT

The `--cgroup-parent` option allows you to set the default cgroup parent to use for containers. If this option is not set, it defaults to `/docker` for fs cgroup driver and `system.slice` for systemd cgroup driver.

If the cgroup has a leading forward slash (`/`), the cgroup is created under the root cgroup, otherwise the cgroup is created under the daemon cgroup.

Assuming the daemon is running in cgroup `daemoncgroup`, `--cgroup-parent=/foobar` creates a cgroup in `/sys/fs/cgroup/memory/foobar`, whereas using `--cgroup-parent=foobar` creates the cgroup in `/sys/fs/cgroup/memory/daemoncgroup/foobar`

The systemd cgroup driver has different rules for `--cgroup-parent`. Systemd represents hierarchy by slice and the name of the slice encodes the location in the tree. So `--cgroup-parent` for systemd cgroups should be a slice name. A name can consist of a dash-separated series of names, which describes the path to the slice from the root slice. For example, `--cgroup-parent=user-a-b.slice` means the memory cgroup for the container is created in `/sys/fs/cgroup/memory/user.slice/user-a.slice/user-a-b.slice/docker-.scope`.

This setting can also be set per container, using the `--cgroup-parent` option on `docker create` and `docker run`, and takes precedence over the `--cgroup-parent` option on the daemon.

#### DAEMON METRICS

The `--metrics-addr` option takes a tcp address to serve the metrics API. This feature is still experimental, therefore, the daemon must be running in experimental mode for this feature to work.

To serve the metrics API on `localhost:9323` you would specify `--metrics-addr 127.0.0.1:9323`, allowing you to make requests on the API at `127.0.0.1:9323/metrics` to receive metrics in the [prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/) format.

Port `9323` is the [default port associated with Docker metrics](https://github.com/prometheus/prometheus/wiki/Default-port-allocations) to avoid collisions with other prometheus exporters and services.

If you are running a prometheus server you can add this address to your scrape configs to have prometheus collect metrics on Docker. For more information on prometheus you can view the website [here](https://prometheus.io/).

```
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['127.0.0.1:9323']
```

#### NODE GENERIC RESOURCES

The `--node-generic-resources` option takes a list of key-value pair (`key=value`) that allows you to advertise user defined resources in a swarm cluster.

The current expected use case is to advertise NVIDIA GPUs so that services requesting `NVIDIA-GPU=[0-16]` can land on a node that has enough GPUs for the task to run.

Example of usage:

```json
{
	"node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"]
}
```

### Daemon configuration file

参数 `--config-file` 允许指定一个 JSON 格式的 daemon 配置文件。这个文件使用与命令行相同的选项名作为键。配置文件中设置的选项不得与通过命令行设置的选项冲突，即使设置的值是相同的，docker deamon 也无法启动。

#### On Linux

配置文件默认存放位置是 `/etc/docker/daemon.json`，参数 `--config-file` 允许指定其他路径。

以下是 Linux 上支持的全部配置选项示例：

```json
{
	"authorization-plugins": [],
	"data-root": "",
	"dns": [],
	"dns-opts": [],
	"dns-search": [],
	"exec-opts": [],
	"exec-root": "",
	"experimental": false,
	"features": {},
	"storage-driver": "",
	"storage-opts": [],
	"labels": [],
	"live-restore": true,
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "10m",
		"max-file":"5",
		"labels": "somelabel",
		"env": "os,customer"
	},
	"mtu": 0,
	"pidfile": "",
	"cluster-store": "",
	"cluster-store-opts": {},
	"cluster-advertise": "",
	"max-concurrent-downloads": 3,
	"max-concurrent-uploads": 5,
	"default-shm-size": "64M",
	"shutdown-timeout": 15,
	"debug": true,
	"hosts": [],
	"log-level": "",
	"tls": true,
	"tlsverify": true,
	"tlscacert": "",
	"tlscert": "",
	"tlskey": "",
	"swarm-default-advertise-addr": "",
	"api-cors-header": "",
	"selinux-enabled": false,
	"userns-remap": "",
	"group": "",
	"cgroup-parent": "",
	"default-ulimits": {
		"nofile": {
			"Name": "nofile",
			"Hard": 64000,
			"Soft": 64000
		}
	},
	"init": false,
	"init-path": "/usr/libexec/docker-init",
	"ipv6": false,
	"iptables": false,
	"ip-forward": false,
	"ip-masq": false,
	"userland-proxy": false,
	"userland-proxy-path": "/usr/libexec/docker-proxy",
	"ip": "0.0.0.0",
	"bridge": "",
	"bip": "",
	"fixed-cidr": "",
	"fixed-cidr-v6": "",
	"default-gateway": "",
	"default-gateway-v6": "",
	"icc": false,
	"raw-logs": false,
	"allow-nondistributable-artifacts": [],
	"registry-mirrors": [],
	"seccomp-profile": "",
	"insecure-registries": [],
	"no-new-privileges": false,
	"default-runtime": "runc",
	"oom-score-adjust": -500,
	"node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],
	"runtimes": {
		"cc-runtime": {
			"path": "/usr/bin/cc-runtime"
		},
		"custom": {
			"path": "/usr/local/bin/my-runc-replacement",
			"runtimeArgs": [
				"--debug"
			]
		}
	},
	"default-address-pools":[
		{"base":"172.80.0.0/16","size":24},
		{"base":"172.90.0.0/16","size":24}
	]
}
```

#### On Windows

配置文件默认存放位置是 `%programdata%\docker\config\daemon.json`，参数 `--config-file` 允许指定其他路径。

以下是 Windows 上支持的全部配置选项示例：

```json
{
    "authorization-plugins": [],
    "data-root": "",
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "experimental": false,
    "features":{},
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "",
    "mtu": 0,
    "pidfile": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "max-concurrent-downloads": 3,
    "max-concurrent-uploads": 5,
    "shutdown-timeout": 15,
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "swarm-default-advertise-addr": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "allow-nondistributable-artifacts": [],
    "registry-mirrors": [],
    "insecure-registries": []
}
```

#### FEATURE OPTIONS

`daemon.json` 文件中的可选字段 `features` 允许用户启用禁用指定的daemon 特性，当前仅支持 `buildkit` 选项，比如 `{"features":{"buildkit": true}}` 启用了 `buildkit` 作为默认的 Docker Image Builder。

#### CONFIGURATION RELOAD BEHAVIOR

以下选项可以在 daemon 运行时配置而不用重启进程：

- `debug`: it changes the daemon to debug mode when set to true.
- `cluster-store`: it reloads the discovery store with the new address.
- `cluster-store-opts`: it uses the new options to reload the discovery store.
- `cluster-advertise`: it modifies the address advertised after reloading.
- `labels`: it replaces the daemon labels with a new set of labels.
- `live-restore`: Enables [keeping containers alive during daemon downtime](https://docs.docker.com/config/containers/live-restore/).
- `max-concurrent-downloads`: it updates the max concurrent downloads for each pull.
- `max-concurrent-uploads`: it updates the max concurrent uploads for each push.
- `default-runtime`: it updates the runtime to be used if not is specified at container creation. It defaults to “default” which is the runtime shipped with the official docker packages.
- `runtimes`: it updates the list of available OCI runtimes that can be used to run containers.
- `authorization-plugin`: it specifies the authorization plugins to use.
- `allow-nondistributable-artifacts`: Replaces the set of registries to which the daemon will push nondistributable artifacts with a new set of registries.
- `insecure-registries`: it replaces the daemon insecure registries with a new set of insecure registries. If some existing insecure registries in daemon’s configuration are not in newly reloaded insecure resgitries, these existing ones will be removed from daemon’s config.
- `registry-mirrors`: it replaces the daemon registry mirrors with a new set of registry mirrors. If some existing registry mirrors in daemon’s configuration are not in newly reloaded registry mirrors, these existing ones will be removed from daemon’s config.
- `shutdown-timeout`: it replaces the daemon’s existing configuration timeout with a new timeout for shutting down all containers.
- `features`: it explicitly enables or disables specific features.

更新和重载相关配置的前提是之前这些选项没有被设置，否则仍然算作冲突，此次的任何修改都不会生效，仍以之前的配置继续运行。

### Run multiple daemons

在一个主机上运行多个 daemon 是一个实验性功能。为了不相互冲突，必须提供不同的参数配置，以下选项是必须为每一个 daemon 配置的：

| 名称 | 描述 |
| ---- | ---- |
| `-b`, `--bridge` string | Attach containers to a network bridge |
| `--exec-root` string | Root directory for execution state files (default "/var/run/docker") |
| `--data-root` string | Root directory of persistent Docker state (default "/var/lib/docker") |
| `-p`, `--pidfile` string | Path to use for daemon PID file (default "/var/run/docker.pid") |
| `-H`, `--host` list | Daemon socket(s) to connect to (default []) |
| `--iptables` | Enable addition of iptables rules (default true) |
| `--config-file` string | Daemon configuration file (default "/etc/docker/daemon.json") |
| `--tlscacert` string | Trust certs signed only by this CA (default "~/.docker/ca.pem") |
| `--tlscert` string | Path to TLS certificate file (default "~/.docker/cert.pem") |
| `--tlskey` string | Path to TLS key file (default ~/.docker/key.pem") |

- The `-b, --bridge=` flag is set to `docker0` as default bridge network. It is created automatically when you install Docker. If you are not using the default, you must create and configure the bridge manually or just set it to ‘none’: `--bridge=none`
- `--exec-root` is the path where the container state is stored. The default value is `/var/run/docker`. Specify the path for your running daemon here.
- `--data-root` is the path where persisted data such as images, volumes, and cluster state are stored. The default value is `/var/lib/docker`. To avoid any conflict with other daemons, set this parameter separately for each daemon.
- `-p, --pidfile=/var/run/docker.pid` is the path where the process ID of the daemon is stored. Specify the path for your pid file here.
- `--host=[]` specifies where the Docker daemon will listen for client connections. If unspecified, it defaults to `/var/run/docker.sock`.
- `--iptables=false` prevents the Docker daemon from adding iptables rules. If multiple daemons manage iptables rules, they may overwrite rules set by another daemon. Be aware that disabling this option requires you to manually add iptables rules to expose container ports. If you prevent Docker from adding iptables rules, Docker will also not add IP masquerading rules, even if you set `--ip-masq` to `true`. Without IP masquerading rules, Docker containers will not be able to connect to external hosts or the internet when using network other than default bridge.
- `--config-file=/etc/docker/daemon.json` is the path where configuration file is stored. You can use it instead of daemon flags. Specify the path for each daemon.
- `--tls*` Docker daemon supports `--tlsverify` mode that enforces encrypted and authenticated remote connections. The `--tls*` options enable use of specific certificates for individual daemons.

无网络的 Docker deamon 示例：

```bash
$ dockerd \
        -H unix:///var/run/docker-bootstrap.sock \
        -p /var/run/docker-bootstrap.pid \
        --iptables=false \
        --ip-masq=false \
        --bridge=none \
        --data-root=/var/lib/docker-bootstrap \
        --exec-root=/var/run/docker-bootstrap
```
