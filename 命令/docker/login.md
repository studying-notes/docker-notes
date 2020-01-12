# `docker login` 命令详解

> 登录 Docker Registry

## 基本信息

### 用法

```
docker login [OPTIONS] [SERVER]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--password , -p` | 密码 |
| `--password-stdin` | 从 STDIN 获取密码 |
| `--username , -u` | 用户名 |

## 扩展信息

### 特权用户条件

`docker login` 必须由 root 权限执行，除了这两种情况：

1. 连接到远程守护进程，例如已配置 `docker engine` 的 `docker-machine`。
2. 当前用户被添加到了 `docker` 用户组。这将影响系统的安全，`docker` 用户组等效于 `root`。

登录成功后，该命令将存储凭据到 Linux 的 `$HOME/.docker/config.json`，或者是 Windows 的 `%USERPROFILE%/.docker/config.json`。

### 凭据存储

Docker Engine 可以将用户凭据保存在外部凭据存储中，例如操作系统的钥匙串中，外部存储比 Docker 配置文件中存储凭证更加安全。为此，Docker 需要一个外部帮助程序来与特定的钥匙串或外部存储进行交互，该助手程序必须在客户端的主机的 `$PATH` 中。

下面列出了当前可用的凭证助手程序：

- D-Bus Secret Service: https://github.com/docker/docker-credential-helpers/releases
- Apple macOS keychain: https://github.com/docker/docker-credential-helpers/releases
- Microsoft Windows Credential Manager: https://github.com/docker/docker-credential-helpers/releases
- [pass](https://www.passwordstore.org/): https://github.com/docker/docker-credential-helpers/releases

#### 配置凭据存储

首先在 `$HOME/.docker/config.json` 中指定凭据存储，配置属性的值为该帮助程序的后缀，即 `docker-credential-` 之后的部分。比如 `docker-credential-osxkeychain`：

```json
{
	"credsStore": "osxkeychain"
}
```

`docker logout` 可以移除凭据。

#### 默认行为

默认情况下，Docker 在每个平台上寻找本机的二进制文件，比如 “osxkeychain” on macOS, “wincred” on windows,  “pass” on Linux。在 Linux 上，如果找不到 “pass” 二进制文件，Docker 会回退寻找“secretservice” 二进制文件。如果当前一个二进制文件都不存在，Docker 就会用 base64 编码，然后把凭据存储在配置文件 `$HOME/.docker/config.json` 中。

#### 凭据助手协议

凭据助手可以是任何实现了该协议的程序或脚本。这个协议深受 Git 的启发，但它在信息共享方面有所不同。

帮助程序始终使用命令中的第一个参数来标识操作。 该参数只有三个可能的值：`store`, `get`, `erase`。

- `store` 命令从标准输入中获取 JSON 格式的信息。该信息带有服务器地址，以标识凭据，用户名以及密码或身份令牌。如果存储的是身份令牌，则用户名应该设置为 `<token>`。

```json
  {
  	"ServerURL": "https://index.docker.io/v1",
  	"Username": "cxfans",
  	"Secret": "password"
  }
```

- `get` 命令从标准输入中获取字符串有效信息，该信息带有 docker engine 所需凭据的服务器地址，例如：`https://index.docker.io/v1`。这个命令将以下 JSON 信息写入 `STDOUT`：

```json
{
	"Username": "cxfans",
	"Secret": "password"
}
```

- `erase` 命令从标准输入中获取字符串有效信息，该信息带有 docker engine 将移除凭据的服务器地址，例如：`https://index.docker.io/v1`。这个命令会把显示是否有问题的错误信息写入 `STDOUT`。

### 凭据助手

凭据助手与上面的凭据存储类似，但充当指定程序来处理特定 Registry 的凭据。默认凭证存储（`credsStore` 或配置文件本身）将不会用于与指定 Registry 的凭证有关的操作。

#### 凭据助手配置

凭据帮助程序的指定方式与 `credsStore` 类似，但允许同时配置多个帮助程序。键指定 Registry 域，值为该帮助程序的后缀，即 `docker-credential-` 之后的部分，即 `docker-credential-` 之后的部分。比如 `docker-credential-osxkeychain`：

```json
{
  "credHelpers": {
    "registry.example.com": "registryhelper",
    "awesomereg.example.org": "hip-star",
    "unicorn.example.io": "vcbait"
  }
}
```

## 示例

### 登录自托管的 Registry

```bash
$ docker login localhost:8080
```

### 从 STDIN 获取密码

指定 `--password-stdin` 参数通过 STDIN 提供密码而不是交互式地手动输入密码。通过这种方法可以防止密码出现在 Shell 的输入历史或日志文件中。

```bash
$ cat ~/password | docker login --username cxfans --password-stdin
```
