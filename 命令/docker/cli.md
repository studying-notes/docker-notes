# Docker 命令行

## 基本信息

运行 `docker`、`docker help`** 或 `docker --help`** 都可以列出帮助信息。

### 用法

```bash
$ docker

Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers
```

译：一个自给自足的容器运行时。大概是指 Docker 拥有一个可以闭环的容器生态。

### 选项

> 英文描述还算容易理解，完美主义者翻译不好，因为表格空间太小，中文看着难受。

| 名称/参数类型 | 默认 | 描述 |
| ---------------------- | ------------- | ------------------------------- |
| **`--config`** string | "/root/.docker" | Location of client config files |
| **`-c, --context`** string | "docker context use" | Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var) |
| **`-D, --debug`** | | Enable debug mode |
| **`--help`** | | Print usage |
| **`-H, --host`** value | [] | Daemon socket(s) to connect to |
| **`-l, --log-level`** string | "info" | Set the logging level ("debug"\|"info"\|"warn"\|"error"\|"fatal") |
| **`--tls`** | | Use TLS; implied by --tlsverify |
| **`--tlscacert`** string | "/root/.docker/ca.pem" | Trust certs signed only by this CA |
| **`--tlscert`** string | "/root/.docker/cert.pem" | Path to TLS certificate file |
| **`--tlskey`** string | "/root/.docker/key.pem" | Path to TLS key file |
| **`--tlsverify`** | | Use TLS and verify the remote |
| **`-v, --version`** | | Print version information and quit |

### 命令

命令部分写成了单独的笔记，见[《Docker 基本命令》](cli-cmds.md)。

## 执行权限

非 root 用户为了避免每次执行命令都输入 `sudo`，可以把当前用户加入“docker”用户组。

```bash
sudo usermod -aG docker your-user
```

## 环境变量

以下是 `docker` 命令行支持的环境变量：

- `DOCKER_API_VERSION` The API version to use (e.g. `1.19`)
- `DOCKER_CONFIG` The location of your client configuration files.
- `DOCKER_CERT_PATH` The location of your authentication keys.
- `DOCKER_CLI_EXPERIMENTAL` Enable experimental features for the cli (e.g. `enabled` or `disabled`)
- `DOCKER_DRIVER` The graph driver to use.
- `DOCKER_HOST` Daemon socket to connect to.
- `DOCKER_NOWARN_KERNEL_VERSION` Prevent warnings that your Linux kernel is unsuitable for Docker.
- `DOCKER_RAMDISK` If set this will disable ‘pivot_root’.
- `DOCKER_STACK_ORCHESTRATOR` Configure the default orchestrator(编排) to use when using `docker stack` management commands.
- `DOCKER_TLS` When set Docker uses TLS.
- `DOCKER_TLS_VERIFY` When set Docker uses TLS and verifies the remote.
- `DOCKER_CONTENT_TRUST` When set Docker uses notary(公证) to sign and verify images. Equates to `--disable-content-trust=false` for build, create, pull, push, run.
- `DOCKER_CONTENT_TRUST_SERVER` The URL of the Notary server to use. This defaults to the same URL as the registry.
- `DOCKER_HIDE_LEGACY_COMMANDS` When set, Docker hides “legacy” top-level commands (such as `docker rm`, and `docker pull`) in `docker help` output, and only `Management commands` per object-type (e.g., `docker container`) are printed. 
- `DOCKER_TMPDIR` Location for temporary Docker files.
- `DOCKER_CONTEXT` Specify the context to use (overrides DOCKER_HOST env var and default context set with “docker context use”)
- `DOCKER_DEFAULT_PLATFORM` Specify the default platform for the commands that take the `--platform` flag.

因为 Docker 是由 Go 开发的，所以也支持任何 Go 支持的环境变量，比如常见的：

- `HTTP_PROXY`
- `HTTPS_PROXY`
- `NO_PROXY`

这些 Go 的环境变量不区分大小写。

**启用命令行的实验性功能**

```bash
$ echo 'export DOCKER_CLI_EXPERIMENTAL=enabled' >> ~/.profile && source ~/.profile
```

## 配置文件

默认情况下， Docker 命令行的配置文件目录 `.docker` 存储在 `$HOME` 目录下。Docker 不推荐修改其中的大部分文件，但是可以通过修改 `config.json` 文件控制 `docker` 命令行为的某些方面。目前为止，一共有3种方法修改 `docker` 命令行为：命令行指定选项参数、设置相关环境变量或者修改 `config.json` 文件。

如果一个环境变量和 `--config` 标志同时被设置，`--config` 标志优先于环境变量。命令行选项将覆盖环境变量，而环境变量将覆盖 `config.json` 文件中指定的属性。（不太理解）

## 改变 `.docker` 目录

为了指定一个不同的目录，可以设置 `DOCKER_CONFIG` 环境变量或 `--config` 命令行选项。如果都设置了，`--config` 命令行选项将覆盖 `DOCKER_CONFIG` 环境变量。下面这个例子指定了 `~/config/config.json` 文件覆盖默认配置执行 `docker ps` 命令。

```bash
$ docker --config ~/config ps
```

但是该标志是暂时性的，仅适用于指定运行的命令，必须通过设置 `DOCKER_CONFIG` 环境变量才能永久性改变配置文件的目录，比如：

```bash
$ echo 'export DOCKER_CONFIG=~/config' >> ~/.profile && source ~/.profile

$ echo 'export DOCKER_CONFIG=~/.docker' >> ~/.profile && source ~/.profile
```

## `config.json` 属性

### 示例

```json
{
 "HttpHeaders": {
 "MyHeader": "MyValue"
 },
 "psFormat": "table {{.ID}}\\t{{.Image}}\\t{{.Command}}\\t{{.Labels}}",
 "imagesFormat": "table {{.ID}}\\t{{.Repository}}\\t{{.Tag}}\\t{{.CreatedAt}}",
 "pluginsFormat": "table {{.ID}}\t{{.Name}}\t{{.Enabled}}",
 "statsFormat": "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}",
 "servicesFormat": "table {{.ID}}\t{{.Name}}\t{{.Mode}}",
 "secretFormat": "table {{.ID}}\t{{.Name}}\t{{.CreatedAt}}\t{{.UpdatedAt}}",
 "configFormat": "table {{.ID}}\t{{.Name}}\t{{.CreatedAt}}\t{{.UpdatedAt}}",
 "serviceInspectFormat": "pretty",
 "nodesFormat": "table {{.ID}}\t{{.Hostname}}\t{{.Availability}}",
 "credsStore": "secretservice",
 "credHelpers": {
 "awesomereg.example.org": "hip-star",
 "unicorn.example.com": "vcbait"
 },
 "stackOrchestrator": "kubernetes",
 "detachKeys": "ctrl-e,e",
 "plugins": {
 "plugin1": {
 "option": "value"
 },
 "plugin2": {
 "anotheroption": "anothervalue",
 "athirdoption": "athirdvalue"
 }
 }
}
```

### 属性解释

- `HttpHeaders` 指定 Docker 客户端发送 HTTP 请求给 Docker 服务端时的头部标识。 Docker 不解析这些头部标识。
- `psFormat` 指定 `docker ps` 默认输出格式。当执行 `docker ps` 命令缺少 `--format` 标志时，Docker 客户端采用该输出格式。若该属性未设置，则采用原始的默认输出格式，格式化属性下同。
- `imagesFormat` 指定 `docker images` 默认输出格式。
- `pluginsFormat` 指定 `docker plugin ls` 默认输出格式。
- `servicesFormat` 指定 `docker service ls` 默认输出格式。
- `serviceInspectFormat` 指定 `docker service inspect` 默认输出格式。
- `statsFormat` 指定 `docker stats` 默认输出格式。
- `secretFormat` 指定 `docker secret ls` 默认输出格式。
- `nodesFormat` 指定 `docker node ls` 默认输出格式。
- `configFormat` 指定 `docker config ls` 默认输出格式。
- `credsStore` 指定一个外部二进制文件作为默认凭据存储。当这个属性被设置时，`docker login` 尝试将凭据存储在 `docker-credential-<value>` 指定的二进制文件中，该文件在 `$PATH` 中可见；否则凭据将被储存在配置的 `auths` 属性中。
- `credHelpers` 指定一组凭据认证助手，在存储和检索特定 registries 的凭据时优先于 `credsStore` 或 `auths`。如果设置了这个属性，则在存储或检索特定 registries 的凭据时将使用二进制 文件`docker-credential-<value>`。
- `stackOrchestrator` 指定运行 `docker stack` 管理命令时使用的默认编排器。有效值为 `"swarm"`、`"kubernetes"` 和 `"all"`. 这个属性可以被 `DOCKER_STACK_ORCHESTRATOR` 环境变量或者命令行的 `--orchestrator` 标志覆盖。
- `detachKeys` 自定义脱离容器的按键序列 `<sequence>` 。默认情况下，一旦附加到一个运行中的容器，用户可以依次按下 `CTRL-p CTRL-q` 键在不终止容器的情况下离开容器。自定义适用于 Docker 客户端启动的所有容器。另外也可以在每个容器的基础上覆盖自定义键或默认键序列，通过在命令行运行时指定 `--detach-keys` 参数即可。`<sequence>` 的格式是一个 `,` 分隔的字符串，比如 `"ctrl-e,e"`，该字符串可以是单个字母 `[a-Z]`，或者 `ctrl-` 结合以下任意一项：
 - `a-z`（单个小写字母）
 - `@`
 - `[`
 - `\\`
 - `_`（下划线）
 - `^`
- `plugins` 包含特定的命令行接口插件。键是插件的名称，值是配置插件功能的特定选项。

### 实验性功能

实验特性（experimental features）提供未来产品功能的预览体验，官方警告不得在生产环境中使用。可以编辑 `config.json` 文件开启实用性功能。

```json
{
 "experimental": "enabled",
 "debug": true
}
```

### 认证服务器（Notary）

如果使用自己的认证服务器和自签名的证书或者内部证书颁发机构，必须在 Docker 配置目录的 `tls/<registry_url>/ca.crt` 中放入凭证。另外，也可以通过将证书添加到系统的根证书颁发机构列表中来全局信任该证书。

## 技巧

1. 单个字符的参数可以合在一起

```bash
$ docker run -i -t --name test busybox sh
# 等价于
$ docker run -it --name test busybox sh
```

2. 布尔选项

布尔选项的形式为 `-d=false`，仅 `-d` 则表示 `-d=true`。

```bash
$ docker run -d
# 等价于
$ docker run -d=true
```

3. 多次指定同一个选项

```bash
$ docker run -a stdin -a stdout -i -t ubuntu /bin/bash
# 等价于
$ docker run -a stdin -a stdout -a stderr ubuntu /bin/ls
```

> **提醒**: 由于 `pty` 实现的限制，`-t` 和 `-a stderr` 不可以同时指定。所有 `stderr` 在 `pty` 模式下简单地被重定向到 `stdout`。

4. 只能指定一次的选项

像 `--name=""` 这样参数必须是一个字符串的，以及想她 `-c=0` 这样参数必须是一个整数的，只能指定一次。
