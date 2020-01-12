# `docker inspect` 命令详解

> 返回有关 Docker 对象的低级信息

## 基本信息

### 用法

```
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--format`, `-f` | 用给定的 Go 模板格式化输出 |
| `--size , -s` | 如果类型为容器，则显示文件总大小 |
| `--type` | 返回指定类型的 JSON |

## 扩展信息

- `docker inspect` 提供了有关由 Docker 控制的构造方面的详细信息。
- 默认情况下，`docker inspect` 将以 JSON 数组呈现结果。

## 示例

### 获取实例的 IP 地址

```bash
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $INSTANCE_ID
```

### 获取实例的 MAC 地址

```bash
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}' $INSTANCE_ID
```

### 获取实例的日志路径

```bash
$ docker inspect --format='{{.LogPath}}' $INSTANCE_ID
```

### 获取实例的镜像名

```bash
$ docker inspect --format='{{.Config.Image}}' $INSTANCE_ID
```

### 列出所有绑定端口

可以遍历数组和结果中的映射以产生简单的文本输出：

```bash
$ docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' $INSTANCE_ID
```

### 找到一个指定端口映射

当字段名称以数字开头时，`.Field` 语法不起作用，但是模板语言的索引函数仍起作用。`.NetworkSettings.Ports` 部分包含内部端口映射到外部地址/端口对象列表的映射。为了获取公共端口数值，可用 `index` 查找特定的端口映射，然后 `index 0` 包含其中的第一个对象。然后，通过 `HostPort` 字段获取公共地址。

```bash
$ docker inspect --format='{{(index (index .NetworkSettings.Ports "8787/tcp") 0).HostPort}}' $INSTANCE_ID
```
### 获取 JSON 格式的小节

```bash
$ docker inspect --format='{{json .Config}}' $INSTANCE_ID
```
