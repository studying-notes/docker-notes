# ` docker version` 命令详解

> 显示 Docker 版本信息

## 基本信息

### 用法

```
docker version [OPTIONS]
```

### 选项

| 名称 | 描述 |
| ------ | ------ |
| `--format , -f` | 用给定的 Go 模板格式化输出 |
| `--kubeconfig` | **Kubernetes** 配置文件 |

## 示例

```bash
$ docker version

Client:
Version:      1.8.0
API version:  1.20
Go version:   go1.4.2
Git commit:   f5bae0a
Built:        Tue Jun 23 17:56:00 UTC 2015
OS/Arch:      linux/amd64

Server:
Version:      1.8.0
API version:  1.20
Go version:   go1.4.2
Git commit:   f5bae0a
Built:        Tue Jun 23 17:56:00 UTC 2015
OS/Arch:      linux/amd64
```

```bash
$ docker version --format '{{.Server.Version}}'

1.8.0
```

```bash
$ docker version --format '{{json .}}'

{"Client":{"Version":"1.8.0","ApiVersion":"1.20","GitCommit":"f5bae0a","GoVersion":"go1.4.2","Os":"linux","Arch":"amd64","BuildTime":"Tue Jun 23 17:56:00 UTC 2015"},"ServerOK":true,"Server":{"Version":"1.8.0","ApiVersion":"1.20","GitCommit":"f5bae0a","GoVersion":"go1.4.2","Os":"linux","Arch":"amd64","KernelVersion":"3.13.2-gentoo","BuildTime":"Tue Jun 23 17:56:00 UTC 2015"}}
```
