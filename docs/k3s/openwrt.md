---
date: 2022-06-27T19:29:27+08:00
author: "Rustle Karl"

title: "在 openwrt 上部署 K3s"
url:  "posts/docker/docs/k3s/openwrt"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

由于 OpenWrt 没有 systemd 和 OpenRC 来管理进程，所以无法通过脚本的方式一键部署，只能二进制部署。

因为 systemd 对于 OpenWrt 来讲过于复杂臃肿，对于几十 MB 甚至几 MB 存储空间的路由器来说实在是装不下。所以 OpenWrt 使用的 init 系统是 procd。

## 下载安装包

https://github.com/k3s-io/k3s/releases

比如

```shell
wget https://github.com/k3s-io/k3s/releases/download/v1.24.1%2Bk3s1/k3s-arm64 -O /usr/bin/k3s
```

```shell
chmod +x /usr/bin/k3s
```

## 启动 K3s 服务节点

```shell
screen -S k3s
```

### 检查配置

```shell
k3s check-config --data-dir /mnt/ssd/k3s
```

发现空间不足。。。

```shell
k3s server --data-dir /mnt/ssd/k3s
```

又发现 cgroup 有问题：

```shell
Failed to find memory cgroup, you may need to add "cgroup_memory=1 cgroup_enable=memory" to your linux cmdline (/boot/cmdline.txt on a Raspberry Pi)
```

放弃折腾。

```shell

```

```shell

```
