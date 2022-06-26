# `docker stop` 命令详解

> 创建一个引用了 SOURCE_IMAGE 的标记 TARGET_IMAGE

## 基本信息

### 用法

```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

## 扩展信息

- 镜像名称由斜杠 `/` 分隔的名称组成，可以选择以 Registry 主机名作为前缀。主机名必须符合标准的 DNS 规则，但不得包含下划线。
- 如果存在主机名，则可以选择后面加上服务端口号。
- 名称组件可能包含小写字母，数字和分隔符。 分隔符定义为一个句点，一个或两个下划线或一个或多个破折号。 名称组件不能以分隔符开头或结尾。
- 标记名必须是有效的 ASCII 码，可以包含大小写字母、数字、下划线、句号和破折号。标记名不能以句号或破折号开头，最多可以包含128个字符。

## 示例

### Tag an image referenced by ID

```bash
$ docker tag 0e5574283393 fedora/httpd:version1.0
```

### Tag an image referenced by Name

```bash
$ docker tag httpd fedora/httpd:version1.0
```

### Tag an image referenced by Name and Tag

```bash
$ docker tag httpd:test fedora/httpd:version1.0.test
```

### Tag an image for a private repository

```bash
$ docker tag 0e5574283393 myregistryhost:5000/fedora/httpd:version1.0
```
