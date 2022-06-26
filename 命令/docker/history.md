# `docker history` 命令详解

> 显示镜像的历史记录

## 基本信息

### 用法

```
docker history [OPTIONS] IMAGE
```

### 选项

| 名称 | 默认 | 描述 |
| ---- | ---- | ---- |
| `--format` | | 用给定的 Go 模板格式化打印镜像 |
| `--human , -H` | `true` | 用友好可读的格式打印大小和日期 |
| `--no-trunc` | | 不截断输出，即完整显示 `IMAGE ID` |
| `--quiet , -q` | | 只显示 `IMAGE ID` |

## 示例

### 镜像构建过程

```bash
$ docker history centos
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
0f3e07c0138f        2 months ago        /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           2 months ago        /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B
<missing>           2 months ago        /bin/sh -c #(nop) ADD file:d6fdacc1972df524a…   220MB
```

### 格式化输出

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :-------------- | :-------------- |
| `.ID` | Image ID |
| `.CreatedSince` | Elapsed time since the image was created if `--human=true`, otherwise timestamp of when image was created |
| `.CreatedAt` | Timestamp of when image was created |
| `.CreatedBy` | Command that was used to create the image |
| `.Size` | Image disk size |
| `.Comment` | Comment for image |

当指定 `--format` 选项时，`history` 命令将完全按照模板声明的方式输出数据，加上 `table` 参数时包含列标题，列项目通过 `:` 分隔。

```bash
$ docker history --format "{table: {.ID}}: {{.CreatedSince}}" centos
0f3e07c0138f: 2 months ago
<missing>: 2 months ago
<missing>: 2 months ago

$ docker history --format "table {{.ID}}: {{.CreatedSince}}" centos
IMAGE: CREATED
0f3e07c0138f: 2 months ago
<missing>: 2 months ago
<missing>: 2 months ago
```
