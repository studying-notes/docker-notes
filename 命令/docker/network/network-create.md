# `docker network create` 命令详解

> 创建网络

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network create [OPTIONS] NETWORK
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--attachable` | | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) 启用手动容器附件 |
| `--aux-address` | | 网络驱动使用的辅助 IPv4 或 IPv6 地址 |
| `--config-from` | | [**API 1.30+**](https://docs.docker.com/engine/api/v1.30/) 用于复制配置的网络 |
| `--config-only` | | [**API 1.30+**](https://docs.docker.com/engine/api/v1.30/) 只创建一个配置网络 |
| `--driver , -d` | `bridge` | 网络管理驱动 |
| `--gateway` | | 主子网网关 IPv4 或 IPv6 地址 |
| `--ingress` | | [**API 1.29+**](https://docs.docker.com/engine/api/v1.29/) 创建 swarm routing-mesh 网络 |
| `--internal` | | 限制外部访问网络 |
| `--ip-range` | | 从一个子范围分配容器 IP |
| `--ipam-driver` | | IP 地址管理驱动 |
| `--ipam-opt` | | 设置 IPAM 驱动程序特定选项 |
| `--ipv6` | | 启用 IPv6 网络 |
| `--label` | | 设置网络的元数据 |
| `--opt , -o` | | 设置驱动程序特定选项 |
| `--scope` | | [**API 1.30+**](https://docs.docker.com/engine/api/v1.30/) 控制网络范围 |
| `--subnet` | | CIDR 格式的子网，表示一个网络段 |

## 扩展信息

- `DRIVER` 接受 `bridge` 或 `overlay` 这些内置的网络驱动。如果安装了第三方的网络驱动，也可以指定 `DRIVER`。
- 默认自动创建 `bridge` 网络。安装 Docker Engine 时，自动创建了一个 `bridge` 网络，这个网络对应 `docker0`，且无法移除，除非卸载 Docker。当用 `docker run` 启动一个新的容器，Docker 会自动把这个容器加入到这个默认的 `bridge` 网络。`bridge` 网络是在单个 Docker Engine 上安装的隔离网络。

```bash
$ docker network create -d bridge my-bridge-network
```

- `overlay` 网络可以跨越多个运行 Docker Engine 的不同主机实现互联，但是创建 `overlay` 网络需要多个前提条件：
    - 访问键值存储数据库。Engine 支持 Consul, Etcd, and ZooKeeper (Distributed store) 键值存储数据库.
    - 连接到键值存储数据库的主机集群。
    - 在群集中的每个主机上正确配置 Engine `daemon`。
- `dockerd` 与 `overlay` 相关的参数有：
    - `--cluster-store`
    - `--cluster-store-opt`
    - `--cluster-advertise`
- 尽管不是必需的，但最好安装 Docker Swarm 来管理组成网络集群。Swarm 提供了完善的发现和服务器管理工​​具。

```bash
$ docker network create -d overlay my-multihost-network
```

### Overlay 网络限制

当您使用默认的基于 VIP 的端点模式创建网络时，您应该创建具有 `/24` 块（默认）的覆盖网络，这将限制您最多 256 个 IP 地址。这样可以解决群集模式的局限性。如果您需要超过 256 个 IP 地址，不要增加 IP 块的大小，您可以使用带有外部负载均衡器的 `dnsrr` 端点模式，或者使用多个更小的覆盖网络。

## 示例

### 运行容器时指定网络

```bash
$ docker run -itd --network=mynet busybox
```

### 指定高级选项

当创建一个网络，Engine 默认会为这个网络创建一个非重叠的子网络。这个子网不是一个已存在网络的细分，它仅用于 IP 寻址的目的。可以用 `--subnet` 参数指定一个子网以覆盖默认。在一个 `bridge` 网络，可以只创建一个子网：

```bash
$ docker network create --driver=bridge --subnet=192.168.0.0/16 br0
```

此外，还可以指定 `--gateway` `--ip-range` 和 `--aux-address` 选项。

```bash
$ docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  br0
```

如果省略 `--gateway` 标志，引擎将从首选池中选择一个。对于 `overlay` 网络和支持创建多个子网的网络驱动程序插件。下面的例子使用了两个 `/25` 子网掩码来遵循网络设计规范，即在一个 `overlay` 网络中不超过 256 个 IP，每个子网有 126 个可用地址。确保子网没有重叠，否则网络创建会失败。

```bash
$ docker network create -d overlay \
  --subnet=192.168.1.0/25 \
  --subnet=192.170.2.0/25 \
  --gateway=192.168.1.100 \
  --gateway=192.170.2.100 \
  --aux-address="my-router=192.168.1.5" --aux-address="my-switch=192.168.1.6" \
  --aux-address="my-printer=192.170.1.5" --aux-address="my-nas=192.170.1.6" \
  my-multihost-network
```

### Bridge 驱动参数

当创建一个自定义网络时，可以向默认的网络驱动传入额外的参数。以下是这些可选选项及其用于 docker0 bridge 的等效标志：

| Option | Equivalent | Description |
| :---------- | :---------- | :---------- |
| `com.docker.network.bridge.name` | - | bridge name to be used when creating the Linux bridge |
| `com.docker.network.bridge.enable_ip_masquerade` | `--ip-masq` | Enable IP masquerading |
| `com.docker.network.bridge.enable_icc` | `--icc` | Enable or Disable Inter Container Connectivity |
| `com.docker.network.bridge.host_binding_ipv4` | `--ip` | Default IP when binding container ports |
| `com.docker.network.driver.mtu` | `--mtu` | Set the containers network MTU |

以下这些参数在执行 `docker network create` 时可以对任意网络驱动有效，功能约等同于 `docker daemon`。

| Argument | Equivalent | Description |
| :----------- | :----------- | :----------- |
| `--gateway` | - | IPv4 or IPv6 Gateway for the master subnet |
| `--ip-range` | `--fixed-cidr` | Allocate IPs from a range |
| `--internal` | - | Restrict external access to the network |
| `--ipv6` | `--ipv6` | Enable IPv6 networking |
| `--subnet` | `--bip` | Subnet for network |

**指定默认绑定 IP 地址**
```bash
$ docker network create \
    -o "com.docker.network.bridge.host_binding_ipv4"="172.19.0.1" \
    simple-network
```

### 网络内部模式

默认情况下，将容器连接到 overlay 网络时，Docker 还将连接一个 bridge 网络以提供外部连接。如果要创建外部隔离的 overlay 网络，则可以指定 `--internal` 选项。

### 网络入口模式

指定 `--ingress` 可以创建一个用于在集群中提供 routing-mesh 的网络。每次只能创建一个入口网络。只有在没有服务依赖于网络的情况下，才可以删除网络。除了`--attachable` 选项，创建 `overlay` 网络时可用的任何选项在创建入口网络时也可用。

```bash
$ docker network create -d overlay \
  --subnet=10.11.0.0/16 \
  --ingress \
  --opt com.docker.network.driver.mtu=9216 \
  --opt encrypted=true \
  my-ingress-network
```
