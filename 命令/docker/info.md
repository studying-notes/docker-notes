# `docker info` 命令详解

> 显示系统范围的信息

## 基本信息

### 用法

```
docker info [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--format`, `-f` | 用给定的 Go 模板格式化输出 |


## 扩展信息

- 既可以指定 `URL`，也可以指定  `-` 从 `STDIN` 获取数据。该 `URL` 可以指向包含文件系统的 `tar` 归档文件，也可以指向 `Docker` 主机上的单个文件。如果指定归档文件，Docker 会将其解压缩到相对于根目录 `/` 的容器中。 如果指定单个文件，则必须指定主机内的完整路径。
- `--change` 参数可以将 `Dockerfile` 的指令应用到已创建的镜像上，支持如下指令：`CMD` | `ENTRYPOINT` | `ENV` | `EXPOSE` | `ONBUILD` | `USER` | `VOLUME` | `WORKDIR`。

## 示例

### 显示输出

```bash
$ docker info
Client:
 Debug Mode: false

Server:
 Containers: 3
  Running: 0
  Paused: 1
  Stopped: 2
 Images: 9
 Server Version: 19.03.5
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.15.0-70-generic
 Operating System: Ubuntu 18.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 985.4MiB
 Name: aliyun
 ID: TTYM:6Z6L:5AYM:CODC:AOIB:6KFK:MJGY:RRT6:HT3I:XYRA:GUSM:3E6J
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  http://f1361db2.m.daocloud.io/
 Live Restore Enabled: false
```

### 显示调试输出

```bash
$ docker -D info
Client:
 Debug Mode: true

Server:
 Containers: 3
  Running: 0
  Paused: 1
  Stopped: 2
 Images: 9
 Server Version: 19.03.5
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.15.0-70-generic
 Operating System: Ubuntu 18.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 985.4MiB
 Name: aliyun
 ID: TTYM:6Z6L:5AYM:CODC:AOIB:6KFK:MJGY:RRT6:HT3I:XYRA:GUSM:3E6J
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  http://f1361db2.m.daocloud.io/
 Live Restore Enabled: false
```

### 格式化输出

```bash
$ docker info --format '{{json .}}'
```
