# ` docker rmi` 命令详解

> 删除一个或多个镜像

## 基本信息

### 用法

```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--force , -f` | 强制删除镜像 |
| `--no-prune` | 不删除未标记 `untagged` 的父镜像 |

## 扩展信息

- 从主机节点删除并取消标记 `tags` 一个或多个镜像。如果镜像有多个标记，将标记作为该命令的参数时，只会删除该标记。如果镜像只有一个标记，则镜像和标记都将被删除。
- 镜像的标识符可以是 `short or long ID`、 `tag`、`digest`。
- 当一个镜像通过标记 `tag` 移除时，摘要信息也会被自动移除。
- 通过 `DIGEST` 拉取的原始镜像没有与之关联的标记 `TAG`，所以无法通过标记删除。

## 示例

### 多种标识方式移除镜像

```bash
# short ID
$ docker rmi fd484f19954f

# tag
$ docker rmi centos:latest

# digest
docker rmi busybox@sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf
```
