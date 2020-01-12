# `docker exec` 命令详解

> 在运行状态的容器中执行命令

## 基本信息

### 用法

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### 选项

| 名称 | 描述 |
| --------------- | ---------------- |
| `--detach , -d` | Detached mode: run command in the background |
| `--detach-keys` | Override the key sequence for detaching a container |
| `--env , -e` | [**API 1.25+**](https://docs.docker.com/engine/api/v1.25/) Set environment variables |
| `--interactive , -i` | Keep STDIN open even if not attached |
| `--privileged` | Give extended privileges to the command |
| `--tty , -t` | Allocate a pseudo-TTY |
| `--user , -u` | Username or UID (format: <name\|uid>[:<group\|gid>]) |
| `--workdir , -w` | [**API 1.35+**](https://docs.docker.com/engine/api/v1.35/) Working directory inside the container |


## 扩展信息

- `COMMAND` 将在容器的默认目录或者构建时 Dockerfile `WORKDIR` 指令指定的自定义目录中运行。
- `COMMAND` 必须是个可执行文件，链接或引用的命令无法工作，比如 `docker exec -ti my_container "echo a && echo b"` 无法工作，但是 `docker exec -ti my_container sh -c "echo a && echo b"` 可以工作。

## 示例

### 附加到运行中的容器

1. 运行一个容器

```bash
$ docker run --name ubuntu_bash --rm -it ubuntu bash
```

2. 另开窗口执行命令

```bash
$ docker exec -d ubuntu_bash touch /tmp/execWorks
```

3. 切回之前的窗口

```bash
$ ls /tmp/
execWorks
```

### 附加到暂停中的容器

```bash
$ docker start test
$ docker pause test

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES
967df26f9447        alpine:latest       "top"               2 hours ago         Up 26 seconds (Paused)                       test

$ docker exec test ls
Error response from daemon: Container test is paused, unpause the container before exec

# 暂停的容器中无法执行 docker exec
```
