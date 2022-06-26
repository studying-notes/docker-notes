---
date: 2020-09-19 21:32:38  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "Dockerfile 详解"  # 文章标题
description: "Docker 可以通过读取 Dockerfile 中的指令来自动构建镜像。Dockerfile 是一个文本文档，它可以包含用户在命令行上调用的所有命令。通过执行 `docker build`，用户可以从 Dockerfile 自动化构建一个镜像。"
url:  "posts/docker/build/dockerfile"  # 设置网页永久链接
tags: [ "docker", "docker-build"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

Docker 可以通过读取 Dockerfile 中的指令来自动构建镜像。Dockerfile 是一个文本文档，它可以包含用户在命令行上调用的所有命令。通过执行 `docker build`，用户可以从 Dockerfile 自动化构建一个镜像。

## 构建方法

参考[《**`docker build` 命令详解**》](../命令/docker/build.md)这篇笔记。

## BuildKit

从 18.09 开始，Docker 支持了新的构建后端 [moby/buildkit](https://github.com/moby/buildkit)。与旧的实现相比，BuildKit 提供了更多的优点，比如：

- 检测并跳过执行过程中未使用的构建阶段
- 并行构建独立的构建阶段
- 两次构建之间仅增量传输构建上下文中更改的文件
- 传输过程中在构建上下文中检测并跳过构建过程未使用的文件
- 使用具有许多新功能的外部 Dockerfile 实现
- 避免其余 API 的副作用
- 优先考虑构建缓存以进行自动清理

为了启用 BuildKit，必须设置环境变量 `DOCKER_BUILDKIT=1`，更多是实验性语法可以参考[官方文档](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/experimental.md)。

## 格式

`Dockerfile` 的基本格式：

```dockerfile
# 注释
INSTRUCTION arguments
# 指令 参数
```

- 指令部分不区分大小写，但是为了方便区分参数，一般约定大写。
- 指令严格按顺序执行。
- `Dockerfile` 指令部分必须以 `FROM` 开头，在 `FROM` 之前可能存在解析器指令、注释和全局作用域的参数。`FROM` 指令指定了 *[Parent Image](../命令/glossary.md)*。
- 除非是有效的解析器指令，否则以 `#` 开头的行都被视为注释，不是出现在开头的 `#` 都被视为参数，注释中不支持换行符。

```dockerfile
# Comment
RUN echo 'we are running some # of cool things'
```

## 解析器指令

- 解析器指令是可选项，一旦设置就会影响 `Dockerfile` 后续行的处理方式。解析器指令不会在构建中增加层，也不会显示为构建步骤。解析器指令可视为一种特殊的注释，格式为 `# directive=value`。单个指令只能出现一次。
- 一旦处理了注释、空行或构建指令，Docker 就不再寻找解析器指令。相反，它将任何格式化为解析器指令的内容视为注释，不再尝试验证它是否可能是解析器指令。因此，所有解析器指令必须位于 `Dockerfile` 最顶层。
- 解析器指令不区分大小写，但是为了方便，一般约定小写，且任何解析器指令之后包括一个空行。解析器指令不支持换行符。

**无效指令：换行**

```dockerfile
# direc \
tive=value
```

**无效指令：指令重复**

```dockerfile
# directive=value1
# directive=value2

FROM ImageName
```

**无效指令：出现在构建指令之后**

```dockerfile
FROM ImageName
# directive=value
```

**无效指令：出现在注释之后**

```dockerfile
# About my dockerfile
# directive=value
FROM ImageName
```

**无效指令：未知指令被视为注释**

```dockerfile
# unknowndirective=value
# knowndirective=value
```

解析器指令中允许非换行空格，以下形式等价：

```dockerfile
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

支持以下两种解析指令：

- `syntax`
- `escape`

### syntax

```dockerfile
# syntax=[remote image reference]
```

例如：

```dockerfile
# syntax=docker/dockerfile
# syntax=docker/dockerfile:1.0
# syntax=docker.io/docker/dockerfile:1
# syntax=docker/dockerfile:1.0.0-experimental
# syntax=example.com/user/repo:tag@sha256:abcdef...
```

这个特性只在启用了 BuildKit 的情况下可用。


```dockerfile
# unknowndirective=value
# knowndirective=value
```




```dockerfile
# unknowndirective=value
# knowndirective=value
```

### escape

```dockerfile
# unknowndirective=value
# knowndirective=value
```


```dockerfile
# unknowndirective=value
# knowndirective=value
```
