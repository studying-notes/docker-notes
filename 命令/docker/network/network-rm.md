# `docker network rm` 命令详解

> 移除一个或多个网络

## 基本信息

### 描述

仅 [**API 1.21**](https://docs.docker.com/engine/api/v1.25/) 及以上版本支持这个命令。

### 用法

```
docker network rm NETWORK [NETWORK...]
```

## 扩展信息

- 在删除网络之前，必须断开连接到它的任何容器。

## 示例

### 删除网络

```bash
$ docker network rm my-network
```

### 删除多个网络

```bash
$ docker network rm 3695c422697f my-network
```

当指定了多个网络，Docker 会尝试一个一个删除，当一个失败，会继续下一个。
