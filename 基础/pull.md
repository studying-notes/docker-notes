---
date: 2020-12-29T13:51:09+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "简述 Image 的获取和存储方式"  # 文章标题
url:  "posts/docker/abc/pull"  # 设置网页链接，默认使用文件名
tags: [ "docker", "turtial"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

> Docker 是如何获取、管理和存储本地镜像文件的？

## Docker 根目录

```bash
$ ls /var/lib/docker
builder/   containers/  network/   plugins/   swarm/  trust/
buildkit/  image/       overlay2/  runtimes/  tmp/    volumes/
```

可以通过修改 `/etc/docker/daemon.json` 改变根目录位置，在文件中增加 `graph` 参数及其路径即可，示例：

```json
{
    "graph": "/root/docker",
}
```

```bash
# 然后重启服务生效
$ systemctl restart docker.service
```

## 镜像引用方式

### Docker Hub 上的官方镜像

1. `ubuntu`：等价于 `ubuntu:latest`，即获取最新的 `ubuntu` 镜像。
2. `ubuntu:16.04`：获取版本号为 `16.04` 的 `ubuntu` 镜像。
3. `ubuntu:@sha256:abcdef`：获取 digest 为 `sha256:abcdef` 的 `ubuntu` 镜像。

### Docker Hub 上的非官方镜像

- `xxx/ubuntu:tag`：xxx 即用户前缀，除了必须加上用户名前缀，引用方式与官方镜像相同。

### 自行搭建 Registry 上的镜像

- `localhost:5000/ubuntu:tag`：localhost:5000 即提供 Registry 服务的 `host:port` ，除此之外引用方式与官方镜像相同，区分用户的功能官方不提供，需自行建设。

### Digest 的必要性

某些镜像在发布之后可能还会做一些更新，比如修复系统漏洞，这时镜像的内容虽然变了，但镜像的名称和标记可能没有改变，用户这时可以通过 `xxx:@sha256abcdf` 的引用方式更新镜像 ，因此有必要引入 digest 参数。

镜像的 digest 就是镜像 Manifest 文件的 sha256 码，当镜像的 layer 发生变化时，layer 的 sha245 码就会改变，而 Manifest 包括了所有 layer 的 sha245 码，所以也会发生变化。这样就保证了一个 digest 唯一对应一个镜像。

## Pull 的大致过程

> 这里的 Docker 是泛指，即对 docker、dockerd 不作区分。对于 layer 来说，镜像配置文件中的 `diff_ids` 是 layer 的 tar 包的 sha256 值，而 Manifest 文件中的 digest 依赖于 `media type`，是 `media type` 文件的 sha256 值，比如 `tar+gzip` 类型，那 digest 就是 layer 的 tar 包经过 gzip 压缩后文件的 sha256 值。

1. Docker 发送镜像引用信息（比如 `ubuntu:latest`）给 Registry 服务器，服务器根据这些信息找到相应镜像的 [Manifest](image.md#Manifest) 文件，然后将 Manifest 文件返回给 Docker；
2. Docker 得到 Manifest 文件后，读取其中的 digest（sha256 码）；
3. 根据 digest 在本地寻找是否存在对应镜像；
4. 不存在则给 Registry 服务器发送信息（包括 digest 的 sha256 码和 `media type`），请求镜像的配置文件（[Image Config](image.md#Image%20Config)）；
5. 根据镜像配置文件中的 `diff_ids`（每一个 sha256 就是每层 layer 对应 tar 包的 sha256 码，tar 包相当于 layer 的原始格式），在本地寻找对应的 layer 是否存在；
6. 不存在则根据 layer 的 sha256 和 `media type` 从服务器拉取相应的 layer（压缩格式）；
7. 解压文件，验证解压后 tar 包的 sha256 是否和配置文件中 `diff_ids` 描述的一致；
8. 根据 Docker 所用的后台文件系统类型，将解压 tar 之后的文件放到指定的目录；
9. 当所有的 layer 都下载完成后，镜像就算完成了下载。

## 镜像的存储方式

1. 拉取一个镜像

```bash
$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
2746a4a261c9: Pull complete
4c1d20cdee96: Pull complete
0d3160e1d0de: Pull complete
c8e37668deea: Pull complete
Digest: sha256:2695d3e10e69cc500a16eae6d6629c803c43ab075fa5ce60813a0fc49c47e859
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

2. 获取 digests 信息

```bash
$ docker images --digests
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
ubuntu              latest              sha256:2695d3e10e69cc500a16eae6d6629c803c43ab075fa5ce60813a0fc49c47e859   549b9b86cb8d        31 hours ago        64.2MB
```

> 提醒：自行制作的本地镜像没有 digest，因为镜像的 manifest 文件内容由 registry 生成。

### repositories.json

`repositories.json` 中记录了和本地 images 相关的 repository 信息，即 Image 标识（引用方式）和 Image ID（sha256）的对应关系，当一个 image 从 registry 上被 pull 下来后，就会自动更新该文件。

```bash
$ cat /var/lib/docker/image/overlay2/repositories.json | python -m json.tool
{
    "Repositories": {
        "ubuntu": {
            "ubuntu:latest": "sha256:549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8",
            "ubuntu@sha256:2695d3e10e69cc500a16eae6d6629c803c43ab075fa5ce60813a0fc49c47e859": "sha256:549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8"
        }
    }
}
```

### 配置文件（Image Config）

Docker 根据第一步得到的 manifest，从 registry 拿到 config 文件，然后保存在 `image/overlay2/imagedb/content/sha256/` 目录下（当前及之后的工作目录都为 `/var/lib/docker`），文件名称就是文件内容的 sha256 码，即 Image ID。

1. 验证 sha256 码

```bash
$ sha256sum image/overlay2/imagedb/content/sha256/549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8

549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8  image/overlay2/imagedb/content/sha256/549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8
```

2. 输出 Image Config

```bash
$ cat image/overlay2/imagedb/content/sha256/549b9b86cb8d75a2b668c21c50ee092716d070f129fd1493f95ab7e43767eab8 | pyt
hon -mjson.tool
{
    ……

    "rootfs": {
        "diff_ids": [
            "sha256:2dc9f76fb25b31e0ae9d36adce713364c682ba0d2fa70756486e5cedfaf40012",
            "sha256:9f3bfcc4a1a8a676da07287a1aa6f2dcc8e869ea6f054c337593481a5bb1345e",
            "sha256:27dd43ea46a831c39d224e7426794145fba953cd7309feccf4d5ea628072f6a2",
            "sha256:918efb8f161b4cbfa560e00e8e0efb737d7a8b00bf91bb77976257cd0014b765"
        ],
        "type": "layers"
    }

    ……
}
```

> 其他信息都与镜像存储无关，这里只关注 `rootfs` 这个参数即可。`diff_ids` 是一个包含了所有 layer 的列表，严格按顺序排列，从上到下依次是从底层到顶层，每一个 sha256 就是每层 layer 对应 tar 包的 sha256 码。

### layer 的 diffid 和 digest 的关系

layer 的 diffid 存在于 Image Config 文件中（`diff_ids` 字段），而 layer 的 digest 存在于 Image Manifest 文件中，它们的映射关系存储于 `image/overlay2/distribution` 目录下：

```bash
$ tree -d image/overlay2/distribution
image/overlay2/distribution
├── diffid-by-digest
│   └── sha256
└── v2metadata-by-diffid
    └── sha256
```

- **diffid-by-digest**：存放 digest 到 diffid 的映射关系。
- **v2metadata-by-diffid**：存放 diffid 到 digest 的映射关系。

以 layer（2dc9……）为例，输出其 digest 信息：

```bash
$ cat image/overlay2/distribution/v2metadata-by-diffid/sha256/2dc9f76fb25b31e0ae9d36adce713364c682ba0d2fa70756486e5cedfaf40012 | python -m json.tool
[
    {
        "Digest": "sha256:2746a4a261c9e18bfd7ff0429c18fd7522acc14fa4c7ec8ab37ba5ebaadbc584",
        "HMAC": "",
        "SourceRepository": "docker.io/library/ubuntu"
    }
]
```

或者根据其 digest 得到 diffid：

```bash
$ cat image/overlay2/distribution/diffid-by-digest/sha256/2746a4a261c9e18bfd7ff0429c18fd7522acc14fa4c7ec8ab37ba5ebaadbc584
sha256:2dc9f76fb25b31e0ae9d36adce713364c682ba0d2fa70756486e5cedfaf40012
```

### layer 的元数据

layer 的属性信息都放在了 `image/overlay2/layerdb/sha256` 目录下，目录名称是 layer 的 `chainid`。

计算 chainid 时，用到了所有下层 layer 的 diffid，从而能保证根据 chainid 得到的 rootfs 是唯一的。计算公式如下：

```
ChainID(L₀) =  DiffID(L₀)
ChainID(L₀|...|Lₙ₋₁|Lₙ) = Digest(ChainID(L₀|...|Lₙ₋₁) + " " + DiffID(Lₙ))
```

详细解释可以参考[官方文档](https://github.com/opencontainers/image-spec/blob/master/config.md#layer-chainid)。

由此可知，最底层的 layer 的 `chainid` 和 `diffid` 相同，为了计算方便，这里以次底层 layer（9f3b……）作为示例：

1. 计算次底层的 `chainid`

```bash
$ echo -n "sha256:2dc9f76fb25b31e0ae9d36adce713364c682ba0d2fa70756486e5cedfaf40012 sha256:9f3bfcc4a1a8a676da07287a1aa6f2dcc8e869ea6f054c337593481a5bb1345e" | sha256sum -
61199b56f34827cbab596c63fd6e0ac0c448faa7e026e330994818190852d479  -
```

2. 根据 `chainid` 输出对应目录的内容

```bash
$ ls image/overlay2/layerdb/sha256/61199b56f34827cbab596c63fd6e0ac0c448faa7e026e330994818190852d479
cache-id  diff  parent  size  tar-split.json.gz

# 每层 layer 都有这样一个对应的文件夹
```

- `cache-id` 是 Docker 拉取 layer 时生成的一个随机 id，指向存放 layer 文件的地方。

```bash
$ cat cache-id
a8ca3aa423165e0a99ee3a75a09c69e0125c64cfdfdf8fc46fb1ec4eeaf7a854
```

- `diff` 文件存放 layer 的 `diffid`。

```bash
$ cat diff
sha256:9f3bfcc4a1a8a676da07287a1aa6f2dcc8e869ea6f054c337593481a5bb1345e
```

- `parent` 文件存放当前 layer 的下层 layer 的 diffid。最底层 layer 不存在这个文件。

```bash
$ cat parent
sha256:2dc9f76fb25b31e0ae9d36adce713364c682ba0d2fa70756486e5cedfaf40012
```

- `size` 当前 layer 的大小，字节。

```bash
$ cat size
987485
```

- `tar-split.json.gz` layer 压缩包的 split 文件，通过这个文件可以还原 layer 的 tar 包，在 `docker save` 命令执行时会用到。

### layer 数据

Docker 根据存储驱动文件系统类型，在 `/var/lib/docker` 目录下创建了对应的子目录，Ubuntu 18.04.3 LTS 下 Docker 的默认文件系统是 overlay2，所以所有 layer 的文件都放在了 `/var/lib/docker/overlay2` 目录下。这里也就只介绍 [overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 方式下的存储结构。

```bash
$ ls /var/lib/docker/overlay2
4fc6d3e8dae68bcdd0e0f5d9155b103e72cdb86622092aea098be02271182801/
6ce87718d6b11e6effcd214227b612981773c4ec1105e944e784965599137d0c/
7ca826cb2bfe0cde6382c593d7c9e65516abfb7673706dc95e75c877e930814e/
a8ca3aa423165e0a99ee3a75a09c69e0125c64cfdfdf8fc46fb1ec4eeaf7a854/
l/
```

`l/` 目录（小写 L）包含 layer 的短标识符作为符号链接。这些标识符是为了避免在 `mount` 命令的参数上达到页面大小限制。

```bash
$ tree /var/lib/docker/overlay2/l
/var/lib/docker/overlay2/l
├── 6KZ4JJ2V4XUZNI3KNJSFL4J4PQ -> ../7ca826cb2bfe0cde6382c593d7c9e65516abfb7673706dc95e75c877e930814e/diff
├── HDWMZOPMK3PYC2PI6XLSZWKBYV -> ../4fc6d3e8dae68bcdd0e0f5d9155b103e72cdb86622092aea098be02271182801/diff
├── LTRH4UVKAGD2C6H3CSKSEIAREO -> ../6ce87718d6b11e6effcd214227b612981773c4ec1105e944e784965599137d0c/diff
└── SK3G4B4L5C6XAXQ4QYIL4TL46W -> ../a8ca3aa423165e0a99ee3a75a09c69e0125c64cfdfdf8fc46fb1ec4eeaf7a854/diff
```

最底层的 layer 目录，一般包含以下三个项目：

```bash
$ ls 7ca826cb2bfe0cde6382c593d7c9e65516abfb7673706dc95e75c877e930814e
committed  diff/  link

# 由于 sha256 命名的文件夹名字过长，以下都省略了
$ cat link
6KZ4JJ2V4XUZNI3KNJSFL4J4PQ

$ ls diff
bin/   etc/   lib64/  opt/   run/   sys/  var/
boot/  home/  media/  proc/  sbin/  tmp/
dev/   lib/   mnt/    root/  srv/   usr/
```

- `link`：短标识符。
- `diff`：包含了 layer 内容的文件夹，上层只存储与下层 layer 有差异的项目。
- `committed`：大概是这一层 layer 做过的 `commit` 信息，**最顶层这个文件可以不存在**。

**容器非 `running` 状态下**，最底层之上的 layer 目录，一般包含以下五个项目：

```bash
$ ls 4fc6d3e8dae68bcdd0e0f5d9155b103e72cdb86622092aea098be02271182801
committed  diff/  link  lower  work/

$ cat lower
l/SK3G4B4L5C6XAXQ4QYIL4TL46W:l/6KZ4JJ2V4XUZNI3KNJSFL4J4PQ

$ ls diff
etc/  sbin/  usr/  var/
```

- `lower`：包含其全部上层 layer（父 layer）的短链接，以“:”分隔，严格按顺序排列，从左到右层次降低。
- `work`：OverlayFS 的 `workdir`。
- `diff`：只存储与下层 layer 有差异的项目。

**容器 `running` 状态下**，Docker 会另外创建一个 layer 目录（容器的 layer，其实不算是镜像的 layer，但就在这里写了），置于最顶层。

1. 运行容器

```bash
$ docker run -it ubuntu bash
```

2. 另开窗口

```bash
$ ls /var/lib/docker/overlay2
4fc6d3e8dae68bcdd0e0f5d9155b103e72cdb86622092aea098be02271182801/
6ce87718d6b11e6effcd214227b612981773c4ec1105e944e784965599137d0c/
76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c/
76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c-init/
7ca826cb2bfe0cde6382c593d7c9e65516abfb7673706dc95e75c877e930814e/
a8ca3aa423165e0a99ee3a75a09c69e0125c64cfdfdf8fc46fb1ec4eeaf7a854/
l/
```

发现多了两个名字相近的文件夹，不过此时的活动文件夹必然不是带 `-init` 后缀的那个。

```bash
$ ls 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c-init/
committed  diff/  link  lower  work/

$ ls 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c/
diff/  link  lower  merged/  work/
```

带 `-init` 后缀的文件夹目录结构与之前所讲的一致，用于容器的初始化工作，另外我猜测是因为配置文件是可以改变容器的参数的，但这种改变不应该写入 layer，所以有了这个文件夹。

```bash
$tree 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c-init/diff
diff
├── dev
│   ├── console
│   ├── pts
│   └── shm
└── etc
    ├── hostname
    ├── hosts
    ├── mtab -> /proc/mounts
    └── resolv.conf
```

而不带 `-init` 后缀的文件夹是**容器 `running` 状态下**的最顶层 layer，相比而言多了一个 `merged` 文件夹。

```bash
$ ls 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c/merged
bin/   dev/  home/  lib64/  mnt/  proc/  run/   srv/  tmp/  var/
boot/  etc/  lib/   media/  opt/  root/  sbin/  sys/  usr/

$ ls 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c/diff
# 空文件夹
```

- `merged`：OverlayFS 的 `merged` 文件夹，即联合挂载点所在的目录，包含了所有 layer 的内容（即 `diff` 文件夹中的项目），最顶层的 layer 的 `diff` 文件夹作为 OverlayFS 的 `upperdir`，其余 layer 的 `diff` 文件夹依层次顺序作为 OverlayFS 的 `lowerdir`。

3. 回到容器，新建文件

```bash
$ echo '{"user": "cxfans"}' > cxfans.json
$ ls 76052dd00a7ad6f5b935483cd52feed00002d1c86a409a258c75c30733e1565c/diff
cxfans.json
```

在最顶层 layer 的 `diff` 文件夹中出现了新建的文件，其他删除文件等操作不再演示，因为至此已经完全知道了 Docker 中 OverlayFS 的应用，可以参考另一篇 [OverlayFS](../原理/联合文件系统/OverlayFS.md) 的笔记。

4. 退出容器

- 退出后之前新产生的两个文件夹依然存在，但 `merged` 文件夹随之消失了，但额外留下了的 `.bash_history` 文件。
- 每次 `docker run` 同一个镜像都将产生与之前不同的这样两个文件夹，因为是不同的容器，容器 ID 是不同的。
- 运行 `docker start <container-id>` 则是重用之前的两个文件夹。
- 运行 `docker rm <container-id>` 后这两个文件夹将被删除。

### Manifest 文件去哪了？

Manifest 包含的内容就是对 config 和 layer 的 sha256 + `media type` 描述，目的就是为了下载 config 和 layer，镜像下载完成后，它的使命也就完成了。另外，这些信息对于镜像的本地管理毫无作用，所以 Docker 没有在本地存储一份 Manifest 文件与之镜像对应。

## 被忽略的两个目录

- **`image/overlay2/imagedb/metadata`**：存放的是本地镜像的一些信息，从服务器上下载的镜像不会存数据到这个目录。
- **`image/overlay2/layerdb/mounts`**: 创建容器时，Docker 会为每个容器在镜像的基础上创建一层新的 layer，里面包含 `/etc/hosts`、`/etc/hostname`、`/etc/resolv.conf` 等文件，创建的这一层 layer 就放在这里。

### 章节导航

- 上一节：[浅析 Runtime 标准](runtime.md)
- 下一节：[浅析 CREATE 执行过程](create.md)
