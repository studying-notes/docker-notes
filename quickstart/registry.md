---
date: 2022-06-26T13:22:19+08:00
author: "Rustle Karl"

title: "Docker 私有仓库搭建"
url:  "posts/docker/quickstart/registry"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

可以通过官方提供的私有仓库镜像`registry`来搭建私有仓库。

关于仓库配置说明请参见[configuration.md](https://github.com/docker/distribution/blob/master/docs/configuration.md)

> ⚠️ 注意：也可以通过部署管理工具 `Harbor` 来部署 `registry`

除了 [Harbor](https://github.com/goharbor/harbor) 还有 [rancher](https://github.com/rancher/rancher) ，这两个项目是开发比较活跃的。

### `registry`

```bash
docker pull registry
```

创建容器并运行，创建成功之后，可访问 `http://192.168.99.100:7000/v2/`，来检查仓库是否正常运行，当返回 `{}` 时，表示部署成功。

```bash
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  registry:2

# 自定义存储位置
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v $HOME/_docker/registry:/var/lib/registry \
  registry:2

docker run -d -p 5000:5000 --restart=always --name registry \
    -v `pwd`/config.yml:/etc/docker/registry/config.yml \
    registry:2
```

推送镜像到私有仓库

```bash
# 从官方仓库拉取一个镜像
docker pull nginx:1.13
# 为镜像 `nginx:1.13` 创建一个新标签 `192.168.31.69:7000/test-nginx:1.13`
docker tag nginx:latest 192.168.31.69:5000/test-nginx:1.13
# 推送到私有仓库中
docker push 192.168.31.69:5000/test-nginx:1.13
# The push refers to a repository [192.168.99.100:7000/test-nginx]
# Get https://192.168.99.100:7000/v1/_ping: http: server gave HTTP response to HTTPS client
```

在推送到的时候报错误，默认是使用 `https` 提交，这个搭建的默认使用的是 `http`，解决方法两个：

1. 创建一个 `https` 映射
2. 将仓库地址加入到不安全的仓库列表中

我们使用第二种方法，加入到不安全的仓库列表中，修改docker配置文件 `vi /etc/docker/daemon.json` 添加 `insecure-registries` 配置信息，如果 [daemon.json](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) 文件不存在可以创建，关键配置项，将仓库将入到不安全的仓库列表中。

```js
{
  "insecure-registries":[
    "192.168.31.69:5000"
  ]
}
```

>  如果是 macOS 可以通过 docker 客户端，`Preferences` => `Advanced` => `添加配置` => `Apply & Restart`，重启docker就可以了。  

重启服务 `service docker restart`，默认情况下 push 是会报如下错误的：

```bash
docker push 192.168.99.100:7000/test-nginx:1.13
# The push refers to a repository [192.168.99.100:7000/test-nginx]
# a1a53f8d99b5: Retrying in 1 second
# ...
# received unexpected HTTP status: 500 Internal Server Error
```

上面错误是 `SELinux` 强制访问控制安全系统，阻止导致的错误，通过下面方法禁用 SELinux 之后就可以 push 了。

```bash
setenforce 0  
getenforce   
# Permissive  
```

```bash
# 停止本地 registry
docker container stop registry
# 要删除容器，请使用 docker container rm
docker container stop registry && docker container rm -v registry
# 自定义存储位置
```

### registry mirror

自建 Docker Registry 缓存镜像。见另一个项目。


### `Harbor`

[部署 registry 管理工具 Harbor](tools/harbor.md)

```shell

```

```shell

```
