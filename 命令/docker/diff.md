# `docker diff` 命令详解

> 检查容器文件系统上文件或目录的更改

## 基本信息

### 用法

```
docker diff CONTAINER
```

## 扩展信息

列出自容器创建以来容器文件系统中已更改的文件和目录，可以跟踪三种不同类型的更改：

| 符号 | 描述 |
| :--- | :--- |
| `A` | 被添加 |
| `D` | 被删除 |
| `C` | 被更改 |

## 示例

```bash
$ docker run -it --name demo ubuntu bash
$ echo cxfans > cxfans.md
$ exit

$ docker diff demo
C /root
A /root/.bash_history
A /cxfans.md
```
