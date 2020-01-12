# `docker network connect` 命令详解

> 将容器连接到网络

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network connect [OPTIONS] NETWORK CONTAINER
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--alias` | 为容器添加网络范围的别名 |
| `--driver-opt` | 网络驱动程序选项 |
| `--ip` | IPv4 地址 (e.g., 172.30.100.104) |
| `--ip6` | IPv6 地址 (e.g., 2001:db8::33) |
| `--link` | 添加连接到另一个容器 |
| `--link-local-ip` | 为容器添加本地连接地址 |


## 扩展信息

- 一旦连接，容器就可以与同一网络中的其他容器通信。

## 示例

### 将正在运行的容器连接到网络

```bash
$ docker network connect multi-host-network container1
```

### 在容器启动时将其连接到网络

```bash
$ docker run -itd --network=multi-host-network busybox
```

### 指定容器将在给定网络上使用的 IP 地址

```bash
$ docker network connect multi-host-network container1
```

### 将正在运行的容器连接到网络

```bash
$ docker network connect multi-host-network container1
```

### --link 参数已弃用

```bash
$ docker network connect --link container1:c1 multi-host-network container2
```

### 为容器创建网络别名

```bash
$ docker network connect --alias db --alias mysql multi-host-network container2
```

### 停止、暂停或重启容器的网络

用户可以暂停、重启和停止连接到网络的容器，容器在运行时会自动连接到其配置的网络。

如果指定了 IP 地址，则在重新启动已停止的容器时将重新申请该 IP 地址。如果 IP 地址不再可用，则容器启动失败。确保 IP 地址可用的一种方法是在创建网络时指定 `--ip-range` 范围，并从该范围之外选择静态 IP 地址。这可以确保在此容器不在网络上时，不会将 IP 地址提供给另一个容器。

```bash
$ docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 multi-host-network
```

```bash
$ docker network connect --ip 172.20.128.2 multi-host-network container2
```

一旦接入网络，容器只能通过另一个容器的 IP 地址或名称通信。对于支持多主机连接的网络，凡是连接到了同一网络，无论是不是由不同的 Docker Engine 启动，都可以通信。

一个容器可以连接到一个或多个网络，但是网络类型必须是不同的。
