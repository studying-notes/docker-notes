# `docker restart` 命令详解

> 重新启动一个或多个容器

## 基本信息

### 用法

```
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--time , -t` | `10` | 在杀死容器之前等待停止的秒数 |

## 示例

```bash
$ docker restart my_container
```
