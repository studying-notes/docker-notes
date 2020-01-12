# `docker events` 命令详解

> 从服务器获取实时事件

## 基本信息

### 用法

```
docker events [OPTIONS]
docker system events [OPTIONS]
```

### 选项

| 名称 | 描述 |
| --------------- | ---------------- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--since` | 显示自给定时间戳以来创建的所有事件 |
| `--until` | 流的方式显示事件直到给定的时间戳为止 |



## 扩展信息

`docker events` 可以从服务器获取实时事件，但这些事件因 Docker 对象类型而异。

### 对象类型

针对不同的对象类型服务器会报告不同的事件。

### CONTAINERS

- `attach`
- `commit`
- `copy`
- `create`
- `destroy`
- `detach`
- `die`
- `exec_create`
- `exec_detach`
- `exec_die`
- `exec_start`
- `export`
- `health_status`
- `kill`
- `oom`
- `pause`
- `rename`
- `resize`
- `restart`
- `start`
- `stop`
- `top`
- `unpause`
- `update`

### IMAGES

- `delete`
- `import`
- `load`
- `pull`
- `push`
- `save`
- `tag`
- `untag`

### PLUGINS

- `enable`
- `disable`
- `install`
- `remove`

### VOLUMES

- `create`
- `destroy`
- `mount`
- `unmount`

### NETWORKS

- `create`
- `connect`
- `destroy`
- `disconnect`
- `remove`

### DAEMONS

- reload

### SERVICES

- `create`
- `remove`
- `update`

### NODES

- `create`
- `remove`
- `update`

### SECRETS

- `create`
- `remove`
- `update`

### CONFIGS

- `create`
- `remove`
- `update`

### 过滤、格式输出

### 根据时间过滤

- 参数 `--since` 和 `--until` 可以是 Unix 时间戳、时间格式化的时间戳或者相对于客户端机器时间的 Go 持续时间字符串（比如 `10m`、`1h30m`，当前时间前推）。
- 不提供 `--since` 参数时只返回新的实时事件。
- 日期格式的时间戳支持的格式包括 `RFC3339Nano`、`RFC3339`、`2006-01-02T15:04:05`、`2006-01-02T15:04:05.999999999`、`2006-01-02Z07:00` 和 `2006-01-02`。若时间戳的末尾不提供 `Z` 或 `+-00:00` 时区偏移，则将默认为客户端所在时区。
- 提供 Unix 时间戳，即 `seconds[.nanoseconds]` 的形式时，其中 `seconds` 是自1970年1月1日0点以来经过的秒数，其中 `[.nanoseconds]` 字段长度不超过 9 位。

### 根据关键词过滤

- `-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。
- 多个过滤条件的关键词相同代表 `or`，即符合其中一个就可以。
- 不同过滤条件的关键词相同代表 `and`，即必须两个都符合。

目前支持的过滤关键词有：

- config (`config==<name or id>`)
- container (`container=<name or id>`)
- daemon (`daemon=<name or id>`)
- event (`event=<event action>`)
- image (`image=<repository or tag>`)
- label (`label=<key>` or `label=<key>=<value>`)
- network (`network=<name or id>`)
- node (`node=<id>`)
- plugin (`plugin=<name or id>`)
- scope (`scope=<local or swarm>`)
- secret (`secret=<name or id>`)
- service (`service=<name or id>`)
- type (`type=<container or image or volume or network or daemon or plugin or service or node or secret or config>`)
- volume (`volume=<name>`)

### 格式化输出

- 当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。
- 若是设置格式为 `{{json .}}`，则事件将作为有效的 `json` 行进行流处理。

## 示例

### 显示实时事件

**Shell 1**：监听事件

```bash
$ docker events
```

**Shell 2**：启动、停止容器

```bash
$ docker create --name test alpine:latest top
$ docker start test
$ docker stop test
```

**Shell 1**：切回，查看事件

```bash
2019-12-27T17:37:59.055480536+08:00 container create 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:03.197040193+08:00 network connect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
```

### 根据时间过滤

即可以用绝对的时间戳，也可以用相对于宿主机的时间过滤。

#### 时间戳基础操作

1. 显示当前时间戳

```bash
$ date +%s
1577440032
```

2. 转换时间戳

```bash
$ date -d '2019-12-27 13:00:00' +"%s"
1577422800

$ date -d '2019-12-27T17:38:04' +"%s"
1577439484
```

#### 几种时间过滤方法

```bash
$ docker events --since 1577439484
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)

$ docker events --since '2019-12-27'
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)

$ docker events --since '2019-12-27T17:38:04'
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)

# 当前时间前推 20min
$ docker events --since '20m'
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)

$ docker events --since '2019-12-27T17:38:04' --until '2019-12-27T17:50:04'
2019-12-27T17:38:04.038766676+08:00 container start 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
2019-12-27T17:38:08.083200314+08:00 container kill 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test, signal=15)
2019-12-27T17:38:08.239312090+08:00 container die 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (exitCode=143, image=alpine:latest, name=test)
2019-12-27T17:38:08.403044374+08:00 network disconnect 722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229 (container=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902, name=bridge, type=bridge)
2019-12-27T17:38:08.470534259+08:00 container stop 967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902 (image=alpine:latest, name=test)
```

### 根据关键词过滤

```bash
$ docker events --filter 'event=stop'
$ docker events --filter 'image=alpine'
$ docker events --filter 'container=test'
$ docker events --filter 'container=test' --filter 'container=d9cdb1525ea8'
$ docker events --filter 'container=test' --filter 'event=stop'
$ docker events --filter 'type=volume'
$ docker events --filter 'type=network'
$ docker events --filter 'container=container_1' --filter 'container=container_2'
$ docker events --filter 'type=volume'
$ docker events --filter 'type=network'
$ docker events --filter 'type=plugin'
$ docker events -f type=service
$ docker events -f type=node
$ docker events -f type=secret
$  docker events -f type=config
$ docker events --filter 'scope=swarm'
```

### 格式化输出

```bash
$ docker events --filter 'type=container' --format 'Type={{.Type}}  Status={{.Status}}  ID={{.ID}}' --since '2019-12-27T17:38:04'
Type=container  Status=start  ID=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902
Type=container  Status=kill  ID=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902
Type=container  Status=die  ID=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902
Type=container  Status=stop  ID=967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902
```

**JSON 格式输出**

```bash
$ docker events --format '{{json .}}' --since '2019-12-27T17:40:04'
{"Type":"network","Action":"disconnect","Actor":{"ID":"722c9eaade1455c2e53f8e66a7e07cdda51287cab8213c905bd699d605d4a229","Attributes":{"container":"967df26f944782134bb086d01c21c9dbf01f8f04ae8a49f87eaa18b524a23902","name":"bridge","type":"bridge"}},"scope":"local","time":1577439488,"timeNano":1577439488403044374}
……
```
