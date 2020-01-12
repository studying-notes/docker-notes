# ` docker stats` 命令详解

> 显示实时的容器资源使用情况统计流

## 基本信息

### 用法

```
docker stats [OPTIONS] [CONTAINER...]
```

### 选项

| 名称 | 描述 |
| ------ | ------ |
| `--all , -a` | 显示全部容器，默认只显示运行状态的容器 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--no-stream` | 禁用流统计信息，仅获取第一个结果 |
| `--no-trunc` | 不截断输出 |

## 扩展信息

- 指定一列用空格分隔的容器标识可以将数据限制到一个或多个特定容器。
- 指定一个停止的容器不返回任何数据。
- 在Linux上，Docker CLI 通过从总内存使用量中减去页面缓存使用率来报告内存使用情况。`/containers/(id)/stats` API 不会执行这种计算，而是提供总的内存使用量和页面缓存中的内存量，以便客户端可以根据需要使用数据。
- `PIDS` 列包含该容器创建的进程和内核线程的数量。线程是 Linux 内核使用的术语。其他等效术语是“轻量级进程”或“内核任务”等。`PIDS` 列中的大量数字与少量进程相结合可能表明容器中的某些内容正在创建许多线程。

## 示例

```bash
docker stats

CONTAINER ID        NAME                                    CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
b95a83497c91        awesome_brattain                        0.28%               5.629MiB / 1.952GiB   0.28%               916B / 0B           147kB / 0B          9
67b2525d8ad1        foobar                                  0.00%               1.727MiB / 1.952GiB   0.09%               2.48kB / 0B         4.11MB / 0B         2
e5c383697914        test-1951.1.kay7x1lh1twk9c0oig50sd5tr   0.00%               196KiB / 1.952GiB     0.01%               71.2kB / 0B         770kB / 0B          1
4bda148efbc0        random.1.vnc8on831idyr42slu578u3cr      0.00%               1.672MiB / 1.952GiB   0.08%               110kB / 0B          578kB / 0B          2
```

| 列名 | 描述 |
| :----------- | :----------- |
| `CONTAINER ID` and `Name` | 标识 |
| `CPU %` and `MEM %` | 容器占用宿主机资源的百分比 |
| `MEM USAGE / LIMIT` | 容器占用的内存总量和上限 |
| `NET I/O` | 在该网络接口上容器已发送和接收的数据总量 |
| `BLOCK I/O` | 宿主机块设备容器读取和写入数据的总量 |
| `PIDs` | 容器已创建的进程线程数量 |

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.Container` | Container name or ID (user input) |
| `.Name` | Container name |
| `.ID` | Container ID |
| `.CPUPerc` | CPU percentage |
| `.MemUsage` | Memory usage |
| `.NetIO` | Network IO |
| `.BlockIO` | Block IO |
| `.MemPerc` | Memory percentage (Not available on Windows) |
| `.PIDs` | Number of PIDs (Not available on Windows) |

```bash
$ docker stats --all --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" fervent_panini 5acfcb1b4fd1 drunk_visvesvaraya big_heisenberg

CONTAINER                CPU %               MEM USAGE / LIMIT
fervent_panini           0.00%               56KiB / 15.57GiB
5acfcb1b4fd1             0.07%               32.86MiB / 15.57GiB
drunk_visvesvaraya       0.00%               0B / 0B
big_heisenberg           0.00%               0B / 0B
```

```bash
$ docker stats --format "{{.Container}}: {{.CPUPerc}}"

09d3bb5b1604: 6.61%
9db7aa4d986d: 9.19%
3f214c61ad1d: 0.00%
```

#### Linux 下的默认格式

```bash
$ docker stats --format "table {{.ID}}\t{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}\t{{.PIDs}}"
```

#### Windows 下的默认格式

```powershell
$ docker stats --format "table {{.ID}}\t{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"
```
