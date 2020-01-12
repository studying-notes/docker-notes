# ` docker rm` 命令详解

> 删除一个或多个容器

## 基本信息

### 用法

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 强制删除运行中的容器 `SIGKILL` |
| `--link , -l` | 移除容器间的网络链接 |
| `--volumes , -v` | 删除与容器关联的卷 |

## 示例

### 移除一个容器

这将删除链接 `/redis` 引用的容器。

```bash
$ docker rm /redis
/redis
```

### 移除容器间的链接

这将移除默认桥接网络上容器之间的基础链接，从而删除两个容器之间的所有网络通信，但不适用于用户自定义的网络。

```bash
$ docker rm --link /webapp/redis
/webapp/redis
```

### 强制删除运行中的容器

```bash
$ docker rm --force redis
redis
```

### 删除全部停止状态的容器

```bash
$ docker rm $(docker ps -aq)
```

### 移除容器及其关联数据卷

```bash
$ docker rm -v redis
redis
```

### 删除容器且选择地删除卷

```bash
$ docker create -v awesome:/cx -v /fans --name hello redis
hello

$ docker rm -v hello
```

在这个示例中，`/cx` 将保持不变，而 `/fans` 将被删除。对于 `--volumes-from` 继承的卷也有着相同的行为。
