# `docker pause` 命令详解

> 列出端口映射或容器的特定映射

## 基本信息

### 用法

```
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

## 示例

### 显示全部端口映射

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                            NAMES
b650456536c7        busybox:latest      top                 54 minutes ago      Up 54 minutes       0.0.0.0:1234->9876/tcp, 0.0.0.0:4321->7890/tcp   test

$ docker port test
7890/tcp -> 0.0.0.0:4321
9876/tcp -> 0.0.0.0:1234

$ docker port test 7890/tcp
0.0.0.0:4321

$ docker port test 7890/udp
2014/06/24 11:53:36 Error: No public port '7890/udp' published for test

$ docker port test 7890
0.0.0.0:4321
```
