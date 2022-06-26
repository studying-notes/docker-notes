---
date: 2022-06-26T16:31:39+08:00
author: "Rustle Karl"

title: "Rancher 容器管理平台"
url:  "posts/docker/tools/rancher"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

[Rancher](https://github.com/rancher/rancher) 是为使用容器的公司打造的容器管理平台。

Rancher 简化了使用 Kubernetes 的流程，开发者可以随处运行 Kubernetes（Run Kubernetes Everywhere），满足 IT 需求规范，赋能 DevOps 团队。

### Rancher 2.5.x 及之后的版本

登录到 Linux 主机，然后运行下面这个非常简洁的安装命令。

与 2.4.x 或之前的版本相比，使用docker run命令安装 Rancher 2.5.x 时，需要添加 `--privileged` 标志变量，启用特权模式安装 Rancher。

这个命令在 Ubuntu 22.04 或者 Fedora 都无法启动：

```
docker run --privileged --rm -p 80:80 -p 443:443 rancher/rancher:latest
```

这个命令可以：

```
docker run --privileged --rm -p 80:80 -p 443:443 oats87/rancher:v2.6.3-cgv2
```

很无语，自相矛盾。官方文档里说只支持 V2，结果事实上只支持 V1。

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest
```

### 使用已有的可信证书

```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v /etc/letsencrypt/live/xxxx.cn/cert.pem:/etc/rancher/ssl/cert.pem \
    -v /etc/letsencrypt/live/xxxx.cn/key.pem:/etc/rancher/ssl/key.pem \
    -v /etc/letsencrypt/live/xxxx.cn/cacerts.pem:/etc/rancher/ssl/cacerts.pem \
    -v $HOME/docker/rancher:/var/lib/rancher \
    --privileged \
    rancher/rancher:latest
```

### 使用 Let's Encrypt 证书

```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.mydomain.com
```

### 持久化数据


```bash
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v $HOME/docker/rancher:/var/lib/rancher \
    --privileged \
    rancher/rancher:latest \
    --acme-domain rancher.mydomain.com
```
