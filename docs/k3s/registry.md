---
date: 2022-06-27T13:50:45+08:00
author: "Rustle Karl"

title: "私有镜像仓库配置参考"
url:  "posts/docker/docs/k3s/registry"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

可以配置 Containerd 连接到私有镜像仓库，并使用它们在节点上拉取私有镜像。

启动时，K3s 会检查 `/etc/rancher/k3s/` 中是否存在 registries.yaml 文件，并指示 containerd 使用文件中定义的镜像仓库。

如果你想使用一个私有的镜像仓库，那么你需要在每个使用镜像仓库的节点上以 root 身份创建这个文件。

请注意，server 节点默认是可以调度的。如果你没有在 server 节点上设置污点，那么将在它们上运行工作负载，请确保在每个 server 节点上创建registries.yaml文件。

Containerd 中的配置可以用于通过 TLS 连接到私有镜像仓库，也可以与启用验证的镜像仓库连接。

## 镜像仓库配置文件

该文件由两大部分组成：

- mirrors
- configs

### Mirrors

Mirrors 是一个用于定义专用镜像仓库的名称和 endpoint 的指令，例如。

```shell
mirrors:
  mycustomreg.com:
    endpoint:
      - "https://mycustomreg.com:5000"
```

```
vi /etc/rancher/k3s/registries.yaml
```

每个 mirror 必须有一个名称和一组 endpoint。当从镜像仓库中拉取镜像时，containerd 会逐一尝试这些 endpoint URL，并使用第一个可用的 endpoint。

#### 重写

每个镜像都可以有一组重写。重写可以根据正则表达式来改变镜像的标签。如果镜像仓库中的组织/项目结构与上游的不同，这很有用。

例如，以下配置将透明地从 registry.example.com:5000/mirrorproject/rancher-images/coredns-coredns:1.6.3 拉取镜像 docker.io/rancher/coredns-coredns:1.6.3：

```shell
mirrors:
  docker.io:
    endpoint:
      - "https://registry.example.com:5000"
    rewrite:
      "^rancher/(.*)": "mirrorproject/rancher-images/$1"
```

镜像仍将以原始名称存储，所以 crictl image ls 将显示 docker.io/rancher/coredns-coredns:1.6.3 在节点上是可用的，即使镜像是以不同的名字从镜像仓库中拉取的。

### Configs

Configs 部分定义了每个 mirror 的 TLS 和凭证配置。对于每个 mirror，你可以定义auth和/或tls。

```shell

```

## 测试

```shell
k3s crictl pull nginx
```

```shell

```
