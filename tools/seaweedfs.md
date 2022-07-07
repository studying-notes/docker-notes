---
date: 2022-06-26T21:35:53+08:00
author: "Rustle Karl"

title: "Seaweedfs 分布式文件系统"
url:  "posts/docker/tools/seaweedfs"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

[SeaweedFS](https://github.com/chrislusf/seaweedfs) 是一个简单且高度可扩展的分布式文件系统。 

有两个目标：存储数十亿个文件！ 快速提供文件！ 

SeaweedFS 实现了一个带有 O(1) 磁盘搜索的对象存储，以及一个带有 POSIX 接口的可选 Filer。

## 安装

```shell
git clone https://github.com/chrislusf/seaweedfs.git
```

```shell
cd seaweedfs/weed && make install
```

```shell
cp /root/go/bin/weed /usr/bin/
```

## 名词

- `master`: 主节点，即集群管理，同时存储文件和 fid 映射关系
- `volume`: 1、文件卷节点，实际存储文件；2、卷，一个存储级别
- `client`: 客户端，该 FS 使用 RESTful 交互，所以客户端都归纳为一类
- `dataCenter`: 数据中心，简称 DC
- `rack`: 机架。一个机架属于特定的数据中心，一个数据中心可以包含多个机架。

⚠️： `volume` 节点不可以被负载均衡，不然会出现上传错误。

## 增加储存节点

```shell
mkdir -p /mnt/ssd/seaweedfs/volume01
```

chmod -R o+w /mnt/ssd/seaweedfs/volume01

```shell
screen -S seaweedfs
```

```shell
weed volume -dir="/mnt/ssd/seaweedfs/volume01" -max=1024 -mserver="192.168.0.16:9333" -port=8081
```

```shell
weed volume -dir="/mnt/hhd/seaweedfs/volume02" -max=1024 -mserver="192.168.0.16:9333" -port=8082
```

```shell
weed volume -dir="/mnt/hhd/seaweedfs/volume03" -max=1024 -mserver="192.168.0.16:9333" -port=8083
```

```shell
screen -r seaweedfs
```

## 写文件

**上传文件**：首先，将HTTP `POST`，`PUT`或`GET`请求发送到 `/dir/assign`以获取fid和卷服务器URL：

```bash
curl http://192.168.0.16:9333/dir/assign
```

```bash
http http://192.168.0.16:9333/dir/assign
```

```json
{
    "count": 1,
    "fid": "7,078309bd40ff",
    "publicUrl": "192.168.0.16:14080",
    "url": "192.168.0.16:14080"
}
```

其次，要存储文件内容，请从响应中向 `URL +'/'+ fid` 发送`HTTP` `POST`请求：

```bash
curl -F file=@/root/get-docker.sh http://192.168.0.16:14080/7,078309bd40ff
```

```json
{"name":"get-docker.sh","size":20009,"eTag":"8ae50fc5"}
```

```bash
curl -F file=@/root/camera.jpg http://192.168.0.16:14080/7,078309bd40ff
```

⚠️：要更新，请发送包含更新文件内容的其他POST请求。

## 删除文件

将HTTP DELETE请求发送到相同的`URL +'/'+ fid` URL：

```bash
curl -X DELETE 192.168.0.16:14080/7,078309bd40ff
```

## 保存文件ID

现在，您可以将fid（本例中为3,01637037d6）保存到数据库字段中。

开头的数字 `3` 表示卷 `ID`。 在逗号之后，它是一个文件密钥 `01` 和一个文件 cookie `637037d6`

卷`id`是无符号的32位整数。 文件密钥是无符号的64位整数。 文件cookie是无符号的32位整数，用于防止URL猜测。

文件密钥和文件 cookie 都以十六进制编码。 您可以以自己的格式存储`<volume id，file key，file cookie>`元组，或者只是将fid存储为字符串。

如果存储为字符串，理论上，您需要`8+1+16+8=33`字节。 如果不是绰绰有余，char(33) 就足够了，因为大多数用法不需要 `2^32`卷。

如果空间确实存在问题，您可以使用自己的格式存储文件 ID。对于卷 id，您需要一个 4 字节整数，对于文件密钥，需要 8 字节长整数，对于文件 cookie，需要 4 字节整数。所以 16 个字节绰绰有余。

## 读取文件

以下是如何呈现 URL 的示例。

首先按文件的 `volumeId` 查找卷服务器的 URL：

```bash
curl "http://192.168.0.16:9333/dir/lookup?volumeId=7"
```

```json
{"volumeOrFileId":"7","locations":[{"url":"192.168.0.16:14080","publicUrl":"192.168.0.16:14080"}]}
```

由于（通常）卷服务器不是太多，并且卷不经常移动，因此您可以在大多数时间缓存结果。 根据复制类型，一个卷可以具有多个副本位置。 只需随机选择一个位置即可阅读。

现在您可以通过 url 获取公共 URL，呈现 URL 或直接从卷服务器读取：

```bash
http://192.168.0.16:14080/7,078309bd40ff.jpg
```

⚠️：请注意，我们在这里添加文件扩展名 `.jpg`。 它是可选的，只是客户端指定文件内容类型的一种方式。

## 挂载为文件系统

https://github.com/chrislusf/seaweedfs/wiki/FUSE-Mount

这个命令是前台命令：

```bash
# assuming you already started weed master, weed volume and filer
weed mount -filer=192.168.0.16:8888 -dir=/mnt/seaweedfs -filer.path=/
```

```bash
screen -S fuse
```

```bash
screen -r fuse
```

读写权限搞不懂。。

## 随机生成文件测试

```bash
seq 100 | xargs -i dd if=/dev/zero of={}.dat bs=10M count=1
```

S3 是压缩储存的，上面的命令储存后比较小。

用随机数据源：

```bash
seq 100 | xargs -i dd if=/dev/urandom of={}.dat bs=10M count=1
```

大了很多。

### 测试

apt install sysbench -y

```bash
sysbench --test=fileio --file-total-size=1G prepare

sysbench --test=fileio --file-total-size=1G --file-test-mode=rndrw --max-time=60 --max-requests=0 --num-threads=1 --file-block-size=1m run
```

## S3 CLI

https://github.com/chrislusf/seaweedfs/wiki/AWS-CLI-with-SeaweedFS

```bash
apt install -y awscli
```

```bash
aws configure
```

```bash
aws configure set default.s3.signature_version s3v4
```

### list buckets

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 ls
```

### make a bucket

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 mb s3://main
```

### list files inside the bucket

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 ls s3://main
```

### add an object

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 cp /etc/passwd s3://main
```

### copy an object

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 cp s3://main/passwd s3://main/passwd.txt
```

### remove an object

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 rm s3://main/passwd
```

# remove a bucket

```bash
aws --endpoint-url http://192.168.0.16:8333 s3 rb s3://main
```

```bash

```
