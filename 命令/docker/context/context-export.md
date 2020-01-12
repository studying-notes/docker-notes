# `docker context export` 命令

> 导出上下文到 tar 或 kubeconfig 文件

## 基本信息

### 用法

```
docker context export [OPTIONS] CONTEXT [FILE|-]
```

### 选项

| 名称 | 描述 |
| ----------- | ----------- |
| `--kubeconfig` | 导出为 kubeconfig 文件 |

## 扩展信息

- 导出的文件可以被 `docker context import` 导入。
- 默认输出文件名是 `.dockercontext` 或 `.kubeconfig`。
- 为了导出到 `STDOUT`，可以执行 `docker context export my-context -`。
