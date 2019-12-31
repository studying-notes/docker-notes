# 通过 runc 运行容器

> 实验系统自带 runc 命令，可能是以前的 docker-runc 现在改名为 runc 了。

## 概述

为了通过 runc 运行，必须拥有符合 OCI 标准的 Bundle 的容器。Bundle 就是运行容器时所需项目的集合。因为官方的 [oci-image-tools](https://github.com/opencontainers/image-tools) 太久不更新了，与最新标准不兼容，无法自制 Bundle，我暂时也找不到其他方法生成。所以只能用 `docker export` 的方法从现有容器导出根文件系统（root filesystem），作为实验所需的 Bundle。

## 实验过程

1. 创建相关文件夹

```bash
$ mkdir container && cd container
$ mkdir rootfs
```

2. 导出根文件系统

```bash
$ docker export $(docker create hello-world) | tar -C rootfs -xvf -
$ ls -l rootfs
total 20
drwxr-xr-x 4 root root 4096 Dec 19 20:32 dev
drwxr-xr-x 2 root root 4096 Dec 19 20:32 etc
-rwxrwxr-x 1 root root 1840 Jan  1  2019 hello
drwxr-xr-x 2 root root 4096 Dec 19 20:32 proc
drwxr-xr-x 2 root root 4096 Dec 19 20:32 sys
```

3. 生成 config.json 文件，然后修改

```bash
$ runc spec

$ sed -i 's/"sh"/"\/hello"/' config.json
```

因为生成的 `config.json` 文件中，容器的默认启动进程为 `sh`，与实验想得到的结果不一致，所以必须进行修改，将 `"args": ["sh"]` 改成 `"args": ["/hello"]`。

4. 用 runc 运行容器

```bash
$ runc run container
```
`container` 是容器别名。然后就输出了 hello-word 那段文字，此处省略了。run 命令可以自动创建、启动容器，在退出之后还会自动删除容器，但实际上它是由多个命令组合而成的。

## 一步一步管理容器

1. 修改 config.json，禁用终端

```bash
$ sed -i 's/"terminal": true/"terminal": false/' config.json
```

这种情况下，容器创建了不是立刻启动的，官方说是在后台启动的，可能是我 Linux 还没学的原因，我看了半天相关的一个 [Issue](https://github.com/opencontainers/runc/issues/1721)，不太理解，但还是记录一下重点，有三种情况会遇到处理 pty 的问题：

- 执行 `runc create` 或 `runc run -d` 命令时，`runc` 进行不会驻留，这意味着 `runc` 无法执行任何 IO 转发，所以必须由调用者设置。限制体现的方式取决于终端设置。
    - `terminal: true` 表示 runc 会设置一个 PTY，但是如果没有另外一个拥有主 PTY 文件描述符的进程，runc 无法退出。这是通过 `--console-socket=` 参数发送文件描述符来完成的。
    - `terminal: false` 表示 PTY 不会被设置，STDIO 文件描述符将被容器继承。换句话说，`0..2` 会被设置为容器的 STDIO。在不够仔细的情况下，这可能会造成安全问题。
- `runc run` 意味着 `runc` 将会驻留进行 IO 转发。在这种情况下，`terminal: true` 和 `terminal: false` 看起来是一样的。

最后提 Issue 的人向官方提交了[文档](https://github.com/opencontainers/runc/blob/master/docs/terminals.md)，官方也 `Merged` 了。总之必须设置 `"terminal": false`，否则无法创建容器。

2. 创建容器

```bash
$ runc create container
```

3. 已创建容器状态列表

```bash
$ runc list
ID          PID         STATUS      BUNDLE            CREATED                          OWNER
container   25067       created     /root/container   2019-12-19T13:17:52.091755831Z   root
```

4. 启动该容器

```bash
$ runc start container
# 输出了那段文字，自动退出
```

5. 容器状态变成了已停止

```bash
$ runc list
ID          PID         STATUS      BUNDLE            CREATED                          OWNER
container   0           stopped     /root/container   2019-12-19T13:17:52.091755831Z   root
```

6. 删除容器

```bash
$ runc delete container
```

### 章节导航

- 上一节：[浅析 Image 标准](image.md)
- 下一节：[浅析 Runtime 标准](runtime.md)
