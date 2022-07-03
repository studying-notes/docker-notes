---
date: 2022-07-03T12:53:21+08:00
author: "Rustle Karl"

title: "02 Prometheus + Grafana实现可视化、告警"
url:  "posts/docker/docs/prometheus/quickstart/02-grafana-quick-start"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

Prometheus UI 提供了快速验证 PromQL 以及临时可视化支持的能力，但其可视化能力却比较弱。一般情况下，我们都用 Grafana 来实现对 Prometheus 的可视化实现。

![img](http://dd-static.jd.com/ddimg/jfs/t1/200784/26/24544/64811/62c1217eE828c7659/a47fc80c10a34a4c.jpg)

## 什么是Grafana

Grafana 是一个用来展示各种各样数据的开源软件，在其官网上用这么一段话来说明其作用。

## 快速入门

我们只需要在 Grafana 上配置一个 Prometheus 的数据源。接着我们就可以配置各种图表，Grafana 就会自动去 Prometheus 拉取数据进行展示。

### 启动服务器

https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/

数据卷方式：

```shell
# create a persistent volume for your data in /var/lib/grafana (database and plugins)
docker volume create grafana-storage

# start grafana
docker run --rm -p 3000:3000 --name=grafana -v grafana-storage:/var/lib/grafana grafana/grafana-enterprise
```

目录挂载方式：

```shell
docker run --restart=always -d --user 0 --volume "/mnt/ssd/dockerd/grafana:/var/lib/grafana" -p 3000:3000 grafana/grafana-enterprise
```

Grafana 默认使用 3000 端口启动，我们访问：[http://localhost:3000](http://localhost:3000/) 查看对应页面。

![img](http://dd-static.jd.com/ddimg/jfs/t1/44920/16/19427/16692/62c1217cEc6e7f106/1a8e4c62d81769db.jpg)

默认的账号密码是 admin/admin，登陆进去后是这样的。

![img](http://dd-static.jd.com/ddimg/jfs/t1/16580/32/16670/29480/62c1217cEc68614ad/17b17001bd1d8ddf.jpg)

### 配置数据源

之后我们去设置菜单添加 Prometheus 数据源：

![img](http://dd-static.jd.com/ddimg/jfs/t1/102989/4/30631/10421/62c1217cEef5f0723/384ea0dda1b76176.jpg)

打开如下图所示：

![img](http://dd-static.jd.com/ddimg/jfs/t1/41044/2/19272/18946/62c1217cE7bc46fa2/7d70307f46cc868c.jpg)

之后输入对应的名字和 URL 地址即可：

![img](http://dd-static.jd.com/ddimg/jfs/t1/190818/7/26434/26289/62c1217dE80ead6c8/c439683442c90cfd.jpg)

这里我们添加了一个名为「Prometheus-1」的数据源，数据获取地址为：[http://localhost:9090。](http://localhost:9090。/)

### 配置面板

在 Grafana 中有「Dashboard」和「Panel」的概念，Dashboard 可以理解成「看板」，而 Panel 可以理解成「图表，一个看看板中包含了无数个图表。例如下图就是一个看板（Dashboard）：

![img](http://dd-static.jd.com/ddimg/jfs/t1/53739/35/20406/82557/62c1217eEe04f208d/52ad1afa71a254c8.jpg)

里面一个个小的图表，就是一个个小的图表（Panel）。

点击「+号」-> 「Dashboard」就可以添加一个大面板。

![img](http://dd-static.jd.com/ddimg/jfs/t1/115745/31/27998/11731/62c1217cE943d3c2d/8ddf2df5c54855a0.jpg)

添加后的面板是空白的，下面我们创建一个图标来显示 CPU 的使用率变化情况。点击右上角的创建图表按钮：

![img](http://dd-static.jd.com/ddimg/jfs/t1/30904/3/16214/13425/62c1217eEf3eda939/6460420e1002e2c7.jpg)

点击创建图表会进入如下界面：

![img](http://dd-static.jd.com/ddimg/jfs/t1/73847/9/19897/58928/62c1217eE55181070/2b7c8eb910993186.jpg)

我们设置好数据源、Metrics数据、图表名称，之后点击右上角的 Apply 按钮即可。保存之后我们就可以在面板中看到机器的 CPU 使用率情况了。

![img](http://dd-static.jd.com/ddimg/jfs/t1/60196/5/19360/21952/62c1217dEcda49755/4120b11159418d35.jpg)

### 邮件通道配置

如果我们要使用 Prometheus 进行监控告警，那么 Grafana 也能够实现。

Grafana 的告警渠道有很多，这里我们以邮件告警为例。

首先需要在 Grafana 的配置文件，默认是 conf/default.ini 文件。

![img](http://dd-static.jd.com/ddimg/jfs/t1/72268/12/19469/8552/62c1217dEb5ef9dc4/c2dacc6653302f90.jpg)

在 default.ini 文件中增加如下配置：

```
[smtp]
enabled = true
host = smtp.exmail.qq.com:465
user = xxx@qq.com
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = xxxx
cert_file =
key_file =
skip_verify = false
from_address = xxx@qq.com  //必须与上面的 user 属性一致
from_name = Grafana
ehlo_identity =
```

host 这里是你邮箱所在运营商的 SMTP 服务器。user 属性是发件人的邮箱地址。password 是发件人邮箱的登陆密码。from_address 与 user 属性一样，都是发件人的邮箱地址。from_name 是发件人的显示名称。

修改完成之后，保存配置文件，之后重启 Grafana。接着通过 Alerting 菜单添加告警渠道。

![img](http://dd-static.jd.com/ddimg/jfs/t1/7925/40/17179/27370/62c1217dE82c1c7a9/486c96fe7c5393f6.jpg)

之后填写「提醒通道」名称、类型，之后点击「Send Test」按钮测试一下。

![img](http://dd-static.jd.com/ddimg/jfs/t1/202394/8/15972/28109/62c1217eE9a0aa6f1/b5fa2b3b90962743.jpg)

正常的话，会受到一封测试邮件，这表明邮件配置已经完成。

此外我们还可以配置 AlertManager、钉钉等其他告警方式，配置的流程都大同小异，这里不再赘述。

### 指标告警配置

配置好邮件发送通道信息后，Grafana 就具备了发送邮件的能力。但是什么时候发送邮件呢？这就需要我们进行指标告警配置了。

我们需要在图表面板设置中设置相关报警信息：

![img](http://dd-static.jd.com/ddimg/jfs/t1/112007/12/28715/29992/62c1217dEf7439385/78948b6265dea0cd.jpg)

这里我配置了 1 分钟内值低于 1，那么就报警，即：1 分钟内挂机了，那么就报警。随后我手动关掉了 NodeExport 节点，过了几分钟我就收到了报警邮件。

![img](http://dd-static.jd.com/ddimg/jfs/t1/159318/17/24378/24838/62c1217dE795e9772/c7c360e4dff073a0.jpg)

***更多关于图表的设置，将在后续文章专门讲述，这里不深入讲解。***

## Grafana模板中心

对于线上监控来讲，如果我们每个面板都需要自己从零开始，那么就太累了。事实上，我们用到的许多监控信息都是类似的。因此 [Grafana官网 - Dashboards 模块](https://grafana.com/grafana/dashboards) 提供了下载 Dashboard 模板的功能。

![img](http://dd-static.jd.com/ddimg/jfs/t1/199781/25/26037/69849/62c1217eEd8aea16d/a9206046ef0ee70b.jpg)

Dashboards 里有许多各种类型的 Dashboard 面板，例如 JVM 监控、MySQL 数据库监控等。你只需找到合适自己的监控面板，之后根据 ID 添加即可。

例如我找到的这个这个面板包含了各种常见的资源监控，例如：CPU、内存等。

![img](http://dd-static.jd.com/ddimg/jfs/t1/130484/9/27584/74039/62c1217dE4a0eebee/4933b6036d3459d8.jpg)

你只需要复制它的 ID 并使用 Grafana 的 import 功能导入即可，如下图所示：

![img](http://dd-static.jd.com/ddimg/jfs/t1/85410/4/26203/8318/62c1217cE8e58d0b0/a1ccb044d6d01273.jpg)

最终的效果如图所示：

![img](http://dd-static.jd.com/ddimg/jfs/t1/17439/22/17605/109544/62c1217fE1638aa65/29b14b8d34aed53d.jpg)

## 深入教程

https://shuyi.tech/tags?tag=prometheus-basic-serial
