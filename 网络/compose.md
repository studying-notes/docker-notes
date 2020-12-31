---
date: 2020-12-31T13:07:27+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker Compose 网络设置"  # 文章标题
# description: "文章描述"
url:  "posts/docker/network/compose"  # 设置网页永久链接
tags: [ "docker", "docker-compose"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 基本概念

默认情况下，Compose 会为我们的应用创建一个网络，服务的每个容器都会加入该网络中。这样，容器就可被该网络中的其他容器访问，不仅如此，该容器还能以服务名称作为 hostname 被其他容器访问。

默认情况下，应用程序的网络名称基于 Compose 的工程名称，而项目名称基于 docker-compose.yml 所在目录的名称。如需修改工程名称，可使用 --project-name 标识或 COMPOSE_PORJECT_NAME 环境变量。

举个例子，假如一个应用程序在名为 myapp 的目录中，并且 docker-compose.yml 如下所示：

```yaml
version: '2'

services:

  web:
    build: .
    ports:
      - "8000:8000"

  db:
    image: postgres
```

当我们运行 `docker-compose up` 时，将会执行以下几步：

- 创建一个名为 myapp_default 的网络；
- 使用 web 服务的配置创建容器，它以 “web” 这个名称加入网络 myapp_default ；
- 使用 db 服务的配置创建容器，它以 “db” 这个名称加入网络 myapp_default。

容器间可使用服务名称（web 或 db）作为 hostname 相互访问。例如，web 这个服务可使用 `postgres://db:5432` 访问 db 容器。

## 更新容器

当服务的配置发生更改时，可使用 `docker-compose up` 命令更新配置。此时，Compose 会删除旧容器并创建新容器。新容器会以不同的 IP 地址加入网络，名称保持不变。任何指向旧容器的连接都会被关闭，容器会重新找到新容器并连接上去。

## links

前文讲过，默认情况下，服务之间可使用服务名称相互访问。links 允许我们定义一个别名，从而使用该别名访问其他服务。举个例子：

```yaml
version: '2'
services:
  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```

这样 web 服务就可使用 db 或 database 作为 hostname 访问 db 服务了。

## 指定自定义网络

一些场景下，默认的网络配置满足不了我们的需求，此时我们可使用 networks 命令自定义网络。networks 命令允许我们创建更加复杂的网络拓扑并指定自定义网络驱动和选项。不仅如此，我们还可使用 networks 将服务连接到不是由 Compose 管理的、外部创建的网络。

如下，我们在其中定义了两个自定义网络。

```yaml
version: '2'

services:
  proxy:
    build: ./proxy
    networks:
      - front
  app:
    build: ./app
    networks:
      - front
      - back
  db:
    image: postgres
    networks:
      - back

networks:
  front:
    # Use a custom driver
    driver: custom-driver-1
  back:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

其中，proxy 服务与 db 服务隔离，两者分别使用自己的网络；app 服务可与两者通信。

由本例不难发现，使用 networks 命令，即可方便实现服务间的网络隔离与连接。

## 配置默认网络

除自定义网络外，我们也可为默认网络自定义配置。

```yaml
version: '2'

services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```

这样，就可为该应用指定自定义的网络驱动。

## 使用已存在的网络

一些场景下，我们并不需要创建新的网络，而只需加入已存在的网络，此时可使用external选项。示例：

```yaml
networks:
  default:
    external:
      name: my-pre-existing-network```
```

## Docker Compose 链接外部容器的几种方式

在 Docker 中，容器之间的链接是一种很常见的操作：它提供了访问其中的某个容器的网络服务而不需要将所需的端口暴露给 Docker Host 主机的功能。Docker Compose 中对该特性的支持同样是很方便的。然而，如果需要链接的容器没有定义在同一个 `docker-compose.yml` 中的时候，这个时候就稍微麻烦复杂了点。

在不使用 Docker Compose 的时候，将两个容器链接起来使用 `--link` 参数，相对来说比较简单，以 `nginx` 镜像为例子：

```yaml
docker run --rm --name test1 -d nginx  # 开启一个实例 test1
docker run --rm --name test2 --link test1 -d nginx # 开启一个实例 test2 并与 test1 建立链接
```

这样，`test2` 与 `test1` 便建立了链接，就可以在 `test2` 中使用访问 `test1` 中的服务了。如果使用 Docker Compose，那么这个事情就更简单了，还是以上面的 `nginx` 镜像为例子，编辑 `docker-compose.yml` 文件为：

```yaml
version: "3"
services:
  test2:
    image: nginx
    depends_on:
      - test1
    links:
      - test1
  test1:
    image: nginx
```

最终效果与使用普通的 Docker 命令 `docker run xxxx` 建立的链接并无区别。这只是一种最为理想的情况。

1. 如果容器没有定义在同一个 `docker-compose.yml` 文件中，应该如何链接它们呢？
2. 如果定义在 `docker-compose.yml` 文件中的容器需要与 `docker run xxx` 启动的容器链接，需要如何处理？

针对这两种典型的情况，下面给出我个人测试可行的办法：

## 方式一：让需要链接的容器同属一个外部网络

我们还是使用 nginx 镜像来模拟这样的一个情景：假设我们需要将两个使用 Docker Compose 管理的 nignx 容器（ `test1` 和 `test2` ）链接起来，使得 `test2` 能够访问 `test1` 中提供的服务，这里我们以能 ping 通为准。

首先，我们定义容器 `test1` 的 `docker-compose.yml` 文件内容为：

```yaml
version: "3"
services:
  test2:
    image: nginx
    container_name: test1
    networks:
      - default
      - app_net
networks:
  app_net:
    external: true
```

`test2` 的 `docker-compose.yml` 和 `test1` 的定义完全一致。

正如你看到的那样，这里两个容器的定义里都使用了同一个外部网络 `app_net`, 因此，我们需要在启动这两个容器之前通过以下命令再创建外部网络：

```
docker network create app_net
```

之后，通过 `docker-compose up -d` 命令启动这两个容器，然后执行 `docker exec -it test2 ping test1`，你将会看到如下的输出：

```shell
docker exec -it test2 ping test1
```

```
PING test1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: icmp_seq=0 ttl=64 time=0.091 ms
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.146 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.150 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.145 ms
64 bytes from 172.18.0.2: icmp_seq=4 ttl=64 time=0.126 ms
64 bytes from 172.18.0.2: icmp_seq=5 ttl=64 time=0.147 ms
```

证明这两个容器是成功链接了，反过来在 `test1` 中 ping`test2` 也是能够正常 ping 通的。

如果我们通过 `docker run --rm --name test3 -d nginx` 这种方式来先启动了一个容器 `test3` 并且没有指定它所属的外部网络，而需要将其与 `test1` 或者 `test2` 链接的话，这个时候手动链接外部网络即可：

```shell
docker network connect app_net test3
```

这样，三个容器都可以相互访问了。

## 方式二：更改需要链接的容器的网络模式

通过更改你想要相互链接的容器的网络模式为 `bridge`, 并指定需要链接的外部容器 `external_links` 即可。与同属外部网络的容器可以相互访问的链接方式一不同，这种方式的访问是单向的。

还是以 nginx 容器镜像为例子，如果容器实例 `nginx1` 需要访问容器实例 `nginx2`，那么 `nginx2` 的 `doker-compose.yml` 定义为：

```yaml
version: "3"
services:
  nginx2:
    image: nginx
    container_name: nginx2
    network_mode: bridge
```

与其对应的，`nginx1` 的 `docker-compose.yml` 定义为：

```yaml
version: "3"
services:
  nginx1:
    image: nginx
    external_links:
      - nginx2
    container_name: nginx1
    network_mode: bridge
```

> 这里的 `external_links` 是不能省略的，而且 `nginx1` 的启动必须要在 `nginx2` 之后，否则可能会报找不到容器 `nginx2` 的错误。

接着我们使用ping来测试下连通性：

```shell
docker exec -it nginx1 ping nginx2  # nginx1 to nginx2
```

```
PING nginx2 (172.17.0.4): 56 data bytes
64 bytes from 172.17.0.4: icmp_seq=0 ttl=64 time=0.141 ms
64 bytes from 172.17.0.4: icmp_seq=1 ttl=64 time=0.139 ms
64 bytes from 172.17.0.4: icmp_seq=2 ttl=64 time=0.145 ms
```

```shell
docker exec -it nginx2 ping nginx1 # nginx2 to nginx1
```

以上也能充分证明这种方式是属于单向联通的。

在实际应用中根据自己的需要灵活的选择这两种链接方式，如果想偷懒的话，大可选择第二种。不过我更推荐第一种，不难看出无论是联通性还是灵活性，较为更改网络模式的第二种都更为友好。
