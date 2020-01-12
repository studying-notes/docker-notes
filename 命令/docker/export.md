# `docker export` 命令详解

> 将容器的文件系统导出为 tar 存档文件

## 基本信息

### 用法

```
docker export [OPTIONS] CONTAINER
```

### 选项

| 名称 | 描述 |
| --------------- | ---------------- |
| `--output , -o` | Write to a file, instead of STDOUT |

## 扩展信息

- 容器挂载卷中的数据不会一起导出。
- 如果卷被挂载在了容器现有目录的上层，则将导出基础目录的内容，而不是卷的内容。

## 示例

以下两个命令等价：

```bash
$ docker export red_panda > latest.tar
$ docker export --output="latest.tar" red_panda
```
