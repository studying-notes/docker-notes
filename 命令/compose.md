---
date: 2020-12-30T15:18:36+08:00  # 创建日期
author: "Rustle Karl"  # 作者

title: "Docker Compose 常用命令集合"  # 文章标题
# description: "文章描述"
url:  "posts/cmd/docker-compose"  # 设置网页永久链接
tags: [ "docker"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 排序优先级
chapter: false  # 设置为章节

index: true  # 是否可以被索引
toc: true  # 是否自动生成目录
draft: false  # 草稿
---

## 构建并运行

### 指定 YML 文件和项目名称

```shell
docker-compose -f docker-compose.yml -p project-name up -d
```


### 进行所需的服务镜像构建

```shell
docker-compose build
```

### 打印出详细的 config 文件

```shell
docker-compose config
```

### 创建容器但是不运行

```shell
docker-compose create
```

### 停掉服务，删除容器，不删除镜像

```shell
docker-compose down
```

### 接受服务之间的互动事件，如进行健康检查等

```shell
docker-compose events
```

### 对某个容器执行命令

```shell
docker-compose exec 容器名称 命令
```

### 对某个服务查看日志

```shell
docker-compose logs -ft mysql
```

### 查看服务状态

```shell
docker-compose ps
```

### 重启服务

```shell
docker-compose restart/start/stop [服务名称]
```

### 运行某个服务

```shell
docker-compose run [服务名称]
```

### 查看服务中使用的镜像

```shell
docker-compose images [服务名称]
```

### 强制停止容器，删除

```shell
docker-compose kill
```

### 删除停止的容器

```shell
docker-compose rm
```
