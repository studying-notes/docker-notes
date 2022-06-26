---
date: 2022-06-26T22:45:08+08:00
author: "Rustle Karl"

title: "K3s 轻量级 Kubernetes"
url:  "posts/docker/tools/k3s"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

> https://github.com/k3s-io/k3s.git

k3s 是经 CNCF 一致性认证的 Kubernetes 发行版，专为物联网及边缘计算设计。

k3s 专为在资源有限的环境中运行 Kubernetes 的研发和运维人员设计，将满足日益增长的在边缘计算环境中运行在 x86、ARM64 和 ARMv7 处理器上的小型、易于管理的 Kubernetes 集群需求。k3s 的发布，为开发者们提供了以“Rancher 2.X + k3s”为核心的从数据中心到云到边到端的 K8S 即服务（Kubernetes-as-a-Service），推动 KubernetesEverywhere。

## 安装

```shell
apt install -y curl
```

```shell
curl -sfL https://get.k3s.io | sh -
```

安装完成之后，服务会自动启动。

```shell
systemctl status k3s
```

确认 K3S 集群是否就绪：

```shell
sudo kubectl get nodes
```

如果得到类似如下的结果：

```shell
NAME          STATUS   ROLES                  AGE   VERSION
ubuntu-vm01   Ready    control-plane,master   12m   v1.23.6+k3s1
```

则说明 K3S 集群启动成功。

## 工作节点加入集群

获取 Token：

```shell
cat /var/lib/rancher/k3s/server/node-token
```

```shell
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.75.137:6443 K3S_TOKEN=$(cat /var/lib/rancher/k3s/server/node-token) sh -
```

> 主机名 hostname 必须不一样，否则加入不了！用 VMware 复制的，主机名可能是一样的！

```shell
K3S_TOKEN=K104534d84a46b60fdc0dca7736f6010f2815ec0e3c3ae7d7c057a73ddfc197829c::server:df37b80fc25d3a2bfa157f3438a29cbf

curl -sfL https://get.k3s.io | K3S_URL=https://192.168.75.137:6443 K3S_TOKEN=$K3S_TOKEN sh -
```

检查是否加入成功：

```shell
sudo kubectl get nodes
```

## 二级

### 三级

```shell

```

```shell

```
