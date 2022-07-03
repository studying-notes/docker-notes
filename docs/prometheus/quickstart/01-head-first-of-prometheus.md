---
date: 2022-07-03T11:08:47+08:00
author: "Rustle Karl"

title: "01 Prometheus 快速入门"
url:  "posts/docker/docs/prometheus/quickstart/01-head-first-of-prometheus"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

## 基本原理

Prometheus的基本架构如下图所示：

![img](http://dd-static.jd.com/ddimg/jfs/t1/180580/37/26363/41345/62c108b1Ebeac3290/50c13d0661d42a1a.jpg)

从上图可以看到，整个 Prometheus 可以分为四大部分，分别是：

- **Prometheus 服务器**

Prometheus Server 是 Prometheus 组件中的核心部分，负责实现对监控数据的获取，存储以及查询。

- **NodeExporter 业务数据源**

业务数据源通过 Pull/Push 两种方式推送数据到 Prometheus Server。

- **AlertManager 报警管理器**

Prometheus 通过配置报警规则，如果符合报警规则，那么就将报警推送到 AlertManager，由其进行报警处理。

- **可视化监控界面**

Prometheus 收集到数据之后，由 WebUI 界面进行可视化图标展示。目前我们可以通过自定义的 API 客户端进行调用数据展示，也可以直接使用 Grafana 解决方案来展示。

简单地说，Prometheus 的实现架构也并不复杂。**其实就是收集数据、处理数据、可视化展示，再进行数据分析进行报警处理。** 但其珍贵之处在于提供了一整套可行的解决方案，并且形成了一整个生态，能够极大地降低我们的研发成本。

![img](http://dd-static.jd.com/ddimg/jfs/t1/60786/16/19286/11138/62c108b1E53b66285/6951168be0feb0b9.jpg)

## 快速入门

下面我们通过一个简单的例子来实现对服务器 CPU、内存等硬件信息的监控。

### 安装运行 Prometheus 服务端

Prometheus 服务端负责数据的收集，因此我们应该首先安装并运行 Prometheus Server。

```
docker run --name prometheus --rm -p 127.0.0.1:9090:9090 prom/prometheus
```

输入 `http://localhost:9090/graph` 可以看到如下页面，这个是 Prometheus 自带的监控管理界面。

![img](http://dd-static.jd.com/ddimg/jfs/t1/215088/11/20828/20893/62c108b1E31d97687/5eadfeb388acd626.jpg)

### 运行 NodeExporter 客户端数据源

https://github.com/prometheus/node_exporter

官方不推荐在 Docker 中运行，因为需要访问主机资源。

https://github.com/cloudalchemy/ansible-node-exporter

通过 Ansible 自动安装。见 `src/docs/prometheus/node_exporter/README.md`

NodeExporter 是 Prometheus 提供的一个可以采集到主机信息的应用程序，它能采集到机器的 CPU、内存、磁盘等信息。

访问 `http://192.168.75.131:19100/` 可以看到以下页面：

![img](http://dd-static.jd.com/ddimg/jfs/t1/66955/32/19818/7833/62c108b2E4666dffd/f2a701165663a485.jpg)

访问 `http://192.168.75.131:19100/metrics`，可以看到当前 node exporter 获取到的当前主机的所有监控数据，如下所示：

![img](http://dd-static.jd.com/ddimg/jfs/t1/73165/21/20119/78633/62c108b2E79d72a38/442bd82823dd4044.jpg)

每一个监控指标之前都会有一段类似于如下形式的信息：

```
# HELP node_cpu Seconds the cpus spent in each mode.
# TYPE node_cpu counter
node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 3.0703125
```

其中 HELP 用于解释当前指标的含义，TYPE 则说明当前指标的数据类型。

在上面的例子中 node_cpu 的注释表明当前指标是 cpu0 上 idle 进程占用 CPU 的总时间，CPU 占用时间是一个只增不减的度量指标，从类型中也可以看出 node_cpu 的数据类型是计数器(counter)，与该指标的实际含义一致。

又例如 node_load1 该指标反映了当前主机在最近一分钟以内的负载情况，系统的负载情况会随系统资源的使用而变化，因此 node_load1 反映的是当前状态，数据可能增加也可能减少，从注释中可以看出当前指标类型为仪表盘(gauge)，与指标反映的实际含义一致。

除了这些以外，在当前页面中根据物理主机系统的不同，你还可能看到如下监控指标：

```
node_boot_time：系统启动时间
node_cpu：系统CPU使用量
nodedisk*：磁盘IO
nodefilesystem*：文件系统用量
node_load1：系统负载
nodememeory*：内存使用量
nodenetwork*：网络带宽
node_time：当前系统时间
go_*：node exporter中go相关指标
process_*：node exporter自身进程相关运行指标
```

### 配置 Prometheus 的监控数据源

现在我们运行了 Prometheus 服务器，也运行了业务数据源 NodeExporter。但此时 Prometheus 还获取不到任何数据，我们还需要配置下 prometheus.yml 文件，让其去拉取 Node Exporter 的数据。

我们配置一下 Prometheus 的配置文件，让 Prometheus 服务器定时去业务数据源拉取数据。编辑prometheus.yml 并在 scrape_configs 节点下添加以下内容:

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "nodes"
    static_configs:
      - targets:
          [
            "192.168.75.131:19100",
            "192.168.75.132:19100",
            "192.168.75.133:19100",
            "192.168.75.134:19100",
            "192.168.75.136:19100",
            "192.168.75.137:19100",
          ]
```

上面配置文件配置了两个任务。一个是名为 prometheus 的任务，其从「localhost:9090」地址读取数据。另一个是名为 nodes 的任务，其从多个地址读取数据。

配置完成后，我们重新启动 Prometheus。

```
docker run --rm -p 9090:9090 -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

### 查询监控数据

配置完 Prometheus 读取的数据源之后，Prometheus 便可以从 Node Exporter 获取到数据了。那么接下来我们如何查看到这些数据呢？答案是：Prometheus UI！

Prometheus UI 是 Prometheus 内置的一个可视化管理界面，我们通过 `http://localhost:9090` 就可以访问到该页面。

通过 Prometheus UI 可以查询 Prometheus 收集到的数据，而 Prometheus 定义了 PromQL 语言来作为查询监控数据的语言，其余 SQL 类似。

接下来我们访问 `http://localhost:9090`，进入到 Prometheus Server。如果输入「up」并且点击执行按钮以后，可以看到如下结果：

![img](http://dd-static.jd.com/ddimg/jfs/t1/221974/25/17310/31770/62c108b1E1958bf0b/e20617c9ff396496.jpg)

可以看到 `Element` 处有几条记录，其中 instance 值为 localhost:8080 的记录，value 是 1，这代表对应应用是存活状态。

```
up{group="production",instance="localhost:8080",job="node"}	1
```

例如查看我们所运行 NodeExporter 节点所在机器的内存使用情况，可以输入 `node_memory_active_bytes/(1024*1024*1024)` 查看。

![img](http://dd-static.jd.com/ddimg/jfs/t1/23619/12/17489/56746/62c108b1E10a73591/209697bc6186db0b.jpg)

查看 NodeExporter 节点所在机器 CPU 1 分钟的负载情况，可以输入 `node_load1` 即可查看。

![img](http://dd-static.jd.com/ddimg/jfs/t1/196717/13/26980/51058/62c108b1Ec7c4dc6d/7d54e916ae631d28.jpg)

到这里，我们基本上为完成了数据的收集过程，即数据从业务侧收集到 Prometheus 侧，并且还学会了如何使用 Prometheus 自带的控制台。
