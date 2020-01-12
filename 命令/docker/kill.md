# `docker kill` 命令详解

> 终止一个或多个正在运行的容器

## 基本信息

### 用法

```
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--signal , -s` | `KILL` | 发送给容器的信号 |

## 扩展信息

- `docker kill` 默认向容器内部的主进程发送 `SIGKILL` 信号，也可以通过 `--signal` 参数指定信号。容器标识可以是 ID、名称或 ID 前几位（不管几位数，能唯一表示一个容器即可）。
- 当 shell 形式的 `ENTRYPOINT` 和 `CMD` 作为 `/bin/sh -c` 的子命令运行时，该子命令无法传递信号。这意味着可执行文件不是容器的 PID 1，不会接收 Unix 信号。

## 示例

### 发送 KILL 信号给容器

```bash
$ docker kill container
```

### 发送自定义信号给容器

```bash
$ docker kill --signal=SIGHUP  container
```

自定义信号既可以是名称，也可以是数字，另外 `SIG` 前缀可以省略，所以以下三者等价：

```bash
$ docker kill --signal=SIGHUP container
$ docker kill --signal=HUP container
$ docker kill --signal=1 container
```

关于更多 Linux 标准信号的信息，可以参考 [`signal(7)`](http://man7.org/linux/man-pages/man7/signal.7.html) 主页列表。
