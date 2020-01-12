# `docker plugin set` 命令

> 更改插件的设置

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin set PLUGIN KEY=VALUE [KEY=VALUE...]
```

## 扩展信息

插件必须是禁用状态，当前支持的设置有：

- env variables
- source of mounts
- path of devices
- args

## 示例

### 改变环境变量

```bash
$ docker plugin inspect -f {{.Settings.Env}} tiborvass/sample-volume-plugin
[DEBUG=0]

$ docker plugin set tiborvass/sample-volume-plugin DEBUG=1

$ docker plugin inspect -f {{.Settings.Env}} tiborvass/sample-volume-plugin
[DEBUG=1]
```

### 更改挂载源

```bash
$ docker plugin inspect -f '{{with $mount := index .Settings.Mounts 0}}{{$mount.Source}}{{end}}' myplugin
/foo

$ docker plugins set myplugin mymount.source=/bar

$ docker plugin inspect -f '{{with $mount := index .Settings.Mounts 0}}{{$mount.Source}}{{end}}' myplugin
/bar
```

### 更改设备路径

```bash
$ docker plugin inspect -f '{{with $device := index .Settings.Devices 0}}{{$device.Path}}{{end}}' myplugin

/dev/foo

$ docker plugins set myplugin mydevice.path=/dev/bar

$ docker plugin inspect -f '{{with $device := index .Settings.Devices 0}}{{$device.Path}}{{end}}' myplugin

/dev/bar
```

### 更改参数的来源

```bash
$ docker plugin inspect -f '{{.Settings.Args}}' myplugin

["foo", "bar"]

$ docker plugins set myplugin myargs="foo bar baz"

$ docker plugin inspect -f '{{.Settings.Args}}' myplugin

["foo", "bar", "baz"]
```
