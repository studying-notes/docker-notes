# `docker trust revoke` 命令

> 删除镜像的信任证书

## 基本信息

### 用法

```
docker trust revoke [OPTIONS] IMAGE[:TAG]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--yes , -y` | 不提示是否确认 |

## 扩展信息

- 从已签名存储库中的标记中删除签名。

## 示例

### 撤消已签名标记的签名

```bash
$ docker trust view example/trust-demo
SIGNED TAG          DIGEST                                                              SIGNERS
red                 852cc04935f930a857b630edc4ed6131e91b22073bcc216698842e44f64d2943    alice
blue                f1c38dbaeeb473c36716f6494d803fbfbe9d8a76916f7c0093f227821e378197    alice, bob

List of signers and their keys for example/trust-demo:

SIGNER              KEYS
alice               05e87edcaecb
bob                 5600f5ab76a2

Administrative keys for example/trust-demo:
Repository Key:	ecc457614c9fc399da523a5f4e24fe306a0a6ee1cc79a10e4555b3c6ab02f71e
Root Key:	3cb2228f6561e58f46dbc4cda4fcaff9d5ef22e865a94636f82450d1d2234949
```

### 撤消已签名标记的签名


```bash
$ docker trust revoke example/trust-demo:red
Enter passphrase for delegation key with ID 27d42a8:
Successfully deleted signature for example/trust-demo:red
```
```bash

```
```bash

```
```bash

```
