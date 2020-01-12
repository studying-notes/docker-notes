# `docker stop` 命令详解

> 停止一个或多个运行中的容器

## 基本信息

### 用法

```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--time , -t` | `10` | 在杀死容器之前等待它停止的秒数 |

## 示例

```bash
$ docker stop my_container
```
