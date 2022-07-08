---
date: 2022-07-07T19:31:17+08:00
author: "Rustle Karl"

title: "MinIO 对象存储"
url:  "posts/docker/docs/minio/README"  # 永久链接
tags: [ "Docker", "MinIO" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

MinIO 是一个基于 Apache License v2.0 开源协议的对象存储服务。它兼容亚马逊 S3 云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从 KB 到最大 TB 不等。

MinIO 除了直接作为对象存储使用，还可以作为云上对象存储服务的网关层，无缝对接到 Amazon S3 等。

## MinIO 的基础概念

### 存储相关的概念

- 一个对象存储在一个 Set
- 一个集群划分为多个 Set
- 一个 Set 中的 Drive 尽可能分布在不同的节点上
- 一个 Set 包含的 Drive 数量是固定的，默认由系统根据集群规模自动计算

| 概念名称 | 对应含义解释                                                 |
| :------- | :----------------------------------------------------------- |
| `Object` | 存储的基本对象；比如文件、图片等等                           |
| `Bucket` | 用于存储 `Object` 的逻辑空间；相互之间互相隔离；类似于系统中的顶层文件夹 |
| `Drive`  | 即存储数据的磁盘；所有的对象数据都会存储在 `Drive` 里面      |
| `Set`    | 即一组 `Drive` 的集合；分布式部署根据集群规模自动划分一个或多个 `Set` |

- **纠删码 - EC - Erasure Code**

`MinIO` 使用 **纠删码和校验和** 机制来保证高可靠性，即便丢失一半数量(`N/2`)的硬盘，仍然可以恢复数据。纠删码是一种恢复丢失和损坏数据的数学算法，`MinIO` 采用 `Reed-Solomon code` 将对象拆分成 `N/2` 数据和 `N/2` 奇偶校验块。这就意味着如果是 `12` 块盘，一个对象会被分成 `6` 个数据块、`6` 个奇偶校验块，你可以丢失任意 `6` 块盘，仍可以从剩下的盘中的数据进行恢复。

纠删码的工作原理和 `RAID` 或者复制不同，像 `RAID6` 可以在损失两块盘的情况下不丢数据，而 `MinIO` 纠删码可以在丢失一半的盘的情况下，仍可以保证数据安全。而且 `MinIO` 纠删码是作用在对象级别，可以一次恢复一个对象，而 `RAID` 是作用在卷级别，数据恢复时间很长。`MinIO` 对每个对象单独编码，存储服务一经部署，通常情况下是不需要更换硬盘或者修复。`MinIO` 纠删码的设计目标是为了性能和尽可能的使用硬件加速。

### 对象的存储形式

文件对象上传到 `MinIO` 上面，会在对应的磁盘当中，以 `Bucket` 名称为目录，文件名称为下一级目录，文件名下是 `part.1` 和 `xl.meta`，前者是编码数据块及检验块，后者是元数据文件。

![6f359a6f7fe4d849a0372ce55d4ecb24.png](http://dd-static.jd.com/ddimg/jfs/t1/115663/13/27965/89821/62c6c55dE33f7314a/2cafed8f9a7cdeb5.png)

## 客户端安装

```bash
cd /tmp
```

```bash
git clone https://github.com/minio/mc.git
```

```bash
cd mc && go build
```

```bash
cp mc /usr/bin/
```

### 命令行

```bash
alias        设置/移除/列出自定义的别名
ls           列出文件和文件夹
mb           创建一个存储桶或一个文件夹
rb           移除一个桶
cp           拷贝文件和对象
mirror       给存储桶和文件夹做镜像
cat          显示文件和对象内容
head         显示对象的第n行
pipe         将一个STDIN重定向到一个对象或者文件或者STDOUT
share        生成用于共享的URL
find         基于参数查找文件
sql          在对象上运行SQL查询
stat         显示对象元信息
mv           移动文件和对象
tree         以树的格式列出桶和对象
du           统计磁盘使用情况
retention    设置对象和桶的保留
legalhold    设置对象的合法持有
diff         对两个文件夹或者存储桶比较差异
rm           删除文件和对象
encrypt      管理桶加密配置
events       管理对象通知
watch        监听文件和对象的事件
undo         取消PUT/DELETE操作
policy       管理访问策略
tag          管理桶和对象的标签
ilm          管理桶的生命周期
version      输出版本信息
replicate    配置服务器端桶复制
admin        管理Minio服务器
update       检查软件更新
```

### 配置

```bash
mc alias set minio http://192.168.0.18:9000 admin password
```

## Python

自带 Python 开发包。

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```
