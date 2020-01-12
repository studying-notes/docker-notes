# `docker wait` 命令详解

> 阻塞直到一个或多个容器停止，然后打印其退出代码

## 基本信息

### 用法

```
docker wait CONTAINER [CONTAINER...]
```

## 示例

1. 后台模式运行一个容器

```bash
$ docker run -dit --name=my_container ubuntu bash
```

2. 阻塞容器

```bash
$ docker wait my_container
```

3. 打开另一个窗口停止容器

```bash
$ docker stop my_container
```

之前的窗口打印了退出代码。
