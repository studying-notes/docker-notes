# ` docker logs` 命令详解

> 提取容器的日志

## 基本信息

### 用法

```
docker logs [OPTIONS] CONTAINER
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--details` | Show extra details provided to logs |
| `--follow , -f` | Follow log output |
| `--since` | Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes) |
| `--tail` | Number of lines to show from the end of the logs (default `all`) |
| `--timestamps , -t` | Show timestamps |
| `--until`| [**API 1.35+**](https://docs.docker.com/engine/api/v1.35/) Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes) |

## 扩展信息

`docker logs` 命令可以批量检索在执行时存在的日志。但是这个命令仅适用于以 `json-file` 或 `journald` 日志记录驱动程序启动的容器。

- `docker logs --follow` 命令会继续从容器 `STDOUT` 和 `STDERR` 的输出新的信息。
- 将 `--tail` 的参数设置为负数或非整数是无效的，会被重置为 `all`。
- `docker logs --timestamps`  的参数是 [RFC3339Nano timestamp](https://golang.org/pkg/time/#pkg-constants) , 比如 `2014-09-16T06:17:46.000000000Z`。为确保时间戳对齐，必要时将在时间戳的纳秒部分填充零。
- `docker logs --details` 命令会输出额外的属性，比如环境变量和创建容器时提供给 `--log-opt` 参数的标签。
- `docker logs --since` 命令只显示给定时间之后产生的容器日志。日期可以是 Unix 时间戳、时间格式化的时间戳或者相对于客户端机器时间的 Go 持续时间字符串（比如 `10m`、`1h30m`，当前时间前推）。日期格式的时间戳支持的格式包括 `RFC3339Nano`、`RFC3339`、`2006-01-02T15:04:05`、`2006-01-02T15:04:05.999999999`、`2006-01-02Z07:00` 和 `2006-01-02`。若时间戳的末尾不提供 `Z` 或 `+-00:00` 时区偏移，则将默认为客户端所在时区。提供 Unix 时间戳，即 `seconds[.nanoseconds]` 的形式时，其中 `seconds` 是自1970年1月1日0点以来经过的秒数，其中 `[.nanoseconds]` 字段长度不超过 9 位。
- `--since` 可以和 `--follow` 或 `--tail` 联合过滤。

## 示例

### 在特定时间点之前检索日志

```bash
$ docker run --name test -d busybox sh -c "while true; do $(echo date); sleep 1; done"

$ date
Tue 14 Nov 2017 16:40:00 CET
$ docker logs -f --until=2s
Tue 14 Nov 2017 16:40:00 CET
Tue 14 Nov 2017 16:40:01 CET
Tue 14 Nov 2017 16:40:02 CET
```
