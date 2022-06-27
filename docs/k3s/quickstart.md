---
date: 2022-06-27T12:36:14+08:00
author: "Rustle Karl"

title: "K3s 快速入门指南"
url:  "posts/docker/docs/k3s/quickstart"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

## 安装脚本

K3s 提供了一个安装脚本，可以方便的在 systemd 或 openrc 的系统上将其作为服务安装。这个脚本可以在 https://get.k3s.io 获得。要使用这种方法安装 K3s，只需运行以下命令：

```shell
curl -sfL https://get.k3s.io | sh -
```

国内用户，可以使用以下方法加速安装：

```shell
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

运行此安装后：

- K3s 服务将被配置为在节点重启后或进程崩溃或被杀死时自动重启。
- 将安装其他实用程序，包括 kubectl、crictl、ctr、k3s-killall.sh 和 k3s-uninstall.sh。
- 将 kubeconfig 文件写入到/etc/rancher/k3s/k3s.yaml，由 K3s 安装的 kubectl 将自动使用该文件

要在工作节点上安装并将它们添加到集群，请使用 K3S_URL 和 K3S_TOKEN 环境变量运行安装脚本。以下示例演示了如何加入 worker 节点：

```shell
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

国内用户，可以使用以下方法加速安装：

```shell
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

设置 K3S_URL 参数会使 K3s 以 worker 模式运行。K3s agent 将在所提供的 URL 上向监听的 K3s 服务器注册。K3S_TOKEN 使用的值存储在你的服务器节点上的

```shell
/var/lib/rancher/k3s/server/node-token
```

路径下。

每台计算机必须具有唯一的主机名。如果您的计算机没有唯一的主机名，请传递 K3S_NODE_NAME 环境变量，并为每个节点提供一个有效且唯一的主机名。

## 卸载

如果您使用安装脚本安装了 K3s，那么在安装过程中会生成一个卸载 K3s 的脚本。

卸载 K3s 会删除集群数据和所有脚本。要使用不同的安装选项重新启动集群，请使用不同的标志重新运行安装脚本。

```shell
/usr/local/bin/k3s-uninstall.sh
```

```shell
/usr/local/bin/k3s-agent-uninstall.sh
```
