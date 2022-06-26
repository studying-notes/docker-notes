---
date: 2020-12-31T13:07:27+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Docker 网络介绍"  # 文章标题
# description: "文章描述"
url:  "posts/docker/network"  # 设置网页永久链接
tags: [ "docker", "docker-cli"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

{{ <link src="posts/docker/命令/docker/network/network"> }}

## Docker 默认网络

在新安装 docker 的主机上执行

```
docker network ls
```

便能看到 docker 默认安装的所有网络，分别是 none 网络、host 网络和 bridge 网络。

### none 网络

none 网络就是什么都没有的网络。挂在这个网络下的容器除了 lo，没有其他任何网卡。容器 run 时，可以通过添加 `–network=none` 参数来指定该容器使用 none 网络。

### host 网络

连接到 host 网络的容器共享 Docker 宿主机的网络栈，即容器的网络配置与 host 宿主机完全一样。可以通过添加 `–network=host` 参数来指定该容器使用 host 网络。

在容器中可以看到 host 的所有网卡，并且连 hostname 也是 host 的。直接使用 Docker host 的网络最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择 host 网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题。

Docker host 的另一个用途是让容器可以直接配置 host 网路。比如某些跨 host 的网络解决方案，其本身也是以容器方式运行的，这些方案需要对网络进行配置。

相当于该容器拥有了 host 主机的网络，那么其 ip 等配置也相同，相当于主机中套了一个与外部一模一样的容器，可以直接通过 host 的 ip 地址来访问该容器。

### bridge 网络

在不指定 –network 参数或者 `–network=bridge` 的情况下创建的容器其网络类型都是 bridge。

Docker 在安装时会在宿主机上创建名为 docker0 的网桥，所谓网桥相当于一个虚拟交换机，如果使用上述两种方式 run 的容器都会挂到 docker0 上。

容器和 docker0 之间通过 veth 进行连接，veth 相当于一根虚拟网线，连接容器和虚拟交换机，这样就使得 docker0 与容器连通了。

## 自定义容器网络

理论上有了上述三种网络已经足够满足普通用户的需求，但是有时候可能用户需要指定自己的网络，以此来适应某些配置，如 ip 地址规划等等。

### 创建自定义网络

Docker 提供三种 user-defined 网络驱动：bridge，overlay 和 macvlan。overlay 和 macvlan 用于创建跨主机的网络。

创建 bridge 自定义网络。命令如下：

```
docker network create -d bridge --subnet 172.10.0.0/24 --gateway 172.10.0.1 my_net
```

-d bridge 表示自定义网络的驱动为 bridge，`–subnet 172.10.0.0/24 –gateway 172.10.0.1` 分别指定网段和网关。这样就创建好了一个自动一网络，可以通过以下命令查看此网络的信息：

```
docker network inspect my_net
```

会得到此网络的配置信息，my_net 是刚刚创建的网络名称，如果为 bridge 就是查看 docker 创建的默认 bridge 网络信息。

每创建一个自定义网络便会在宿主机中创建一个网桥（docker0 是创建的默认网桥，其实原理是一致的，而且也是对等的。）。名字为 br-<网络短ID>，可以通过 brctl show 命令查看全部网桥信息。

docker 的自定义网络与 OpenStack 中的网络信息倒是基本一致。所以一通百通，只要 docker 的明白了，所有虚拟化甚至实体的网络也就基本都搞清楚了。

### 使用自定义网络

通过以下命令为容器指定自定义网络：

```
docker run -it --network my_net --ip 172.10.0.3 busybox
```

其实这与使用 docker 默认网络是一致的，都是添加 –network 参数参数，此处也添加了–ip 参数来指定容器的 ip 地址。

## 不同容器之间的连通性

同一个网络（默认网络或者自定义网络）下的容器之间是能 ping 通的，但是不同网络之间的容器由于网络独立性的要求是无法 ping 通的。原因是 iptables-save DROP 掉了 docker 之间的网络，大概如下：

```
-A DOCKER-ISOLATION -i docker0 -o br-ac4fe2d72b18 -j DROP
-A DOCKER-ISOLATION -i br-ac4fe2d72b18 -o docker0 -j DROP
-A DOCKER-ISOLATION -i br-62f17c363f02 -o br-ac4fe2d72b18 -j DROP
-A DOCKER-ISOLATION -i br-ac4fe2d72b18 -o br-62f17c363f02 -j DROP
-A DOCKER-ISOLATION -i br-62f17c363f02 -o docker0 -j DROP
-A DOCKER-ISOLATION -i docker0 -o br-62f17c363f02 -j DROP
```

那么如何让不同网络之间的docker通信呢？接下来介绍容器间通信的三种方式。

### 3.1 IP 通信

IP通信就是直接用IP地址来进行通信，根据上面的分析需要保证两个容器处于同一个网络，那么如果不在同一个网络如何处理呢？

如果是实体机我们很容易理解，只需要为其中一台服务器添加一块网卡连接到另一个网络就可以了。容器同理，只需要为其中一个容器添加另外一个容器的网络就可以了。使用如下命令：

```
docker network connect my_net httpd
```

connect命令能够为httpd容器再添加一个my_net网络（假设httpd原来只有默认的bridge网络）。这样上面创建的busybox容器就能与此次connect的httpd容器进行通信。

### 3.2 Docker DNS Server

通过 IP 访问容器虽然满足了通信的需求，但还是不够灵活。因为我们在部署应用之前可能无法确定IP，部署之后再指定要访问的IP会比较麻烦。对于这个问题，可以通过docker自带的DNS服务解决。

从Docker 1.10 版本开始，docker daemon 实现了一个内嵌的DNS [server](https://www.centos.bz/tag/server/)，使容器可以直接通过“容器名”通信。

方法很简单，只要在启动时用–name为容器命名就可以了。

下面的命令启动两个容器bbox1和bbox2：

```
docker run -it --network=my_net --name=bbox1 busybox
docker run -it --network=my_net --name=bbox2 busybox
```

然后，bbox2就可以直接ping到bbox1了，但是使用docker DNS有个限制，只能在user-defined网络中使用。默认的bridge网络是无法使用的。

### 3.3 joined 容器

joined 容器是另一种实现容器间通信的方式。joined 容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined容器之间可以通过127.0.0.1直接通信。host网络使得容器与宿主机共用同一个网络，而jointed是使得两个容器共用同一个网络。

请看下面的例子：

先创建一个httpd容器，名字为web1。

```
docker run -d -it --name=web1 httpd
```

然后创建busybox容器并通过–network=[container](https://www.centos.bz/tag/container/):web1指定jointed容器为web1：

```
docker run -it --network=container:web1 busybox
```

这样busybox和web1的网卡mac地址与IP完全一样，它们共享了相同的网络栈。busybox 可以直接用127.0.0.1访问web1的http服务。

其实也很容易理解，之前的–network参数指定了默认网络或者自定义网络，而此处是指定了一个容器，那么当然意思就是使用这个容器的网络。这也有点类似上一篇文章讲到的共享存储。

joined 容器非常适合以下场景：

- 不同容器中的程序希望通过loopback高效快速地通信，比如web server与app server。



- 希望监控其他容器的网络流量，比如运行在独立容器中的网络监控程序。

其实就是应用于即需要独立而又需要两个容器网络高度一致的场景。

### 3.4 容器与外部网络的连通性

- 容器访问外部网络

容器默认是能访问外部网络的。通过NAT，docker实现了容器对外网（此处外网不一定是互联网）的访问。

- 外部网络访问容器

通过端口映射的方式实现外部网络访问容器，即通过-p参数实现将容器的端口映射到外部端口。
