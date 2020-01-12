# `docker attach` 命令详解

> 将本地标准输入，输出和错误流附加到正在运行的容器

## 基本信息

### 用法

```
docker attach [OPTIONS] CONTAINER
```

### 选项

| 名称 | 默认值 | 描述 |
| --------------- | ------- | --------------------------------------------------- |
| `--detach-keys` | | Override the key sequence for detaching a container |
| `--no-stdin` | | Do not attach STDIN |
| `--sig-proxy` | `true` | Proxy all received signals to the process |

## 扩展信息

- `docker attach` 将本地标准输入，输出和错误流（或组合）附加到正在运行的容器，然后可以看到 `ENTRYPOINT/CMD` 的输出或在终端中交互式地控制它。实际上，当前进程可能根本不与终端交互，此时看起来就像是被挂起了。

- 可以从宿主机的不同会话同时附加到同一个进程。
- 按 `CTRL-c` 停止容器，这个组合键会发送 `SIGKILL` 给容器。当 `--sig-proxy=true` 时，`CTRL-c` 会发送 `SIGINT` 给容器。若是执行参数中有 `-it`，则默认情况下按 `CTRL-p CTRL-q` 组合键可以退出容器，容器运行终止。

- 当设置了 `-t` 参数后，禁止 `docker attach` 重定向容器的标准输入文件。

- 通过 `docker attach` 连接到容器的标准输入输出时，Docker 会分配约 1MB 的内存缓冲区来最大化程序的吞吐量。当该缓冲区已满时，API 连接的速度将开始对进程输出的写入速度产生影响。因此，不建议运行性能关键型程序，这些程序通过缓慢的客户端连接将在前台生成大量输出。

## 自定义退出组合键

默认情况下，一旦附加到一个运行中的容器，用户可以依次按下 `CTRL-p CTRL-q` 键退出容器。

通过在命令行运行时指定 `--detach-keys=<sequence>` 参数可以自定义该组合键，`<sequence>` 的格式是一个 `,` 分隔的字符串，比如 `"ctrl-e,e"`，该字符串可以是单个字母 `[a-Z]`，或者 `ctrl-` 结合以下任意一项：

- `a-z`（单个小写字母）
- `@`
- `[`
- `\\`
- `_`（下划线）
- `^`

像 `a`, `ctrl-a`, `X`, 或 `ctrl-\\` 都是有效的组合键，不过通过命令行设置都是临时的，永久性设置可以参考[配置文件部分](cli.md#配置文件)。

## 示例

### 自定义退出组合键

```bash
$ docker run -dit --name demo ubuntu bash
$ docker attach demo --detach-keys=ctrl-d
# 按下 ctrl-d
read escape sequence
```
