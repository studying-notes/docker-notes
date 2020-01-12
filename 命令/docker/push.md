# `docker push` 命令详解

> 将镜像或存储库推送到 Registry

## 基本信息

### 用法

```
docker push [OPTIONS] NAME[:TAG]
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--disable-content-trust` | `true` | 跳过镜像签名 |

## 扩展信息

- `CTRL-c` 可以取消上传；在 `docker push` 期间会出现进度条，显示未压缩的大小，但实际推送的数据量将在发送前进行压缩，因此进度条不能反映上传的大小。
- 默认情况下，Docker 守护进程将会同时上传一个镜像的 5 层数据，可以通过 `--max-concurrent-uploads` 守护进程选项手动修改。
