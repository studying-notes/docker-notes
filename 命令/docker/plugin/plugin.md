# `docker plugin` 命令

> 管理插件

## 基本信息

### 描述

仅 [**API 1.25**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

### 用法

```
docker plugin COMMAND
```

### 命令

| 命令 | 描述 |
| ------------- | ------------- |
| [docker plugin create](https://docs.docker.com/engine/reference/commandline/plugin_create/) | 从一个 rootfs 中创建一个插件及其配置。插件数据目录必须包含 config.json 和 rootfs 文件夹 |
| [docker plugin disable](https://docs.docker.com/engine/reference/commandline/plugin_disable/) | 禁用一个插件 |
| [docker plugin enable](https://docs.docker.com/engine/reference/commandline/plugin_enable/) | 启用一个插件 |
| [docker plugin inspect](https://docs.docker.com/engine/reference/commandline/plugin_inspect/) | 显示一个或多个插件的详细信息 |
| [docker plugin install](https://docs.docker.com/engine/reference/commandline/plugin_install/) | 安装一个插件 |
| [docker plugin ls](https://docs.docker.com/engine/reference/commandline/plugin_ls/) | 列出插件 |
| [docker plugin push](https://docs.docker.com/engine/reference/commandline/plugin_push/) | 上传一个插件到 Registry |
| [docker plugin rm](https://docs.docker.com/engine/reference/commandline/plugin_rm/) | 删除一个或多个插件 |
| [docker plugin set](https://docs.docker.com/engine/reference/commandline/plugin_set/) | 更改插件的设置 |
| [docker plugin upgrade](https://docs.docker.com/engine/reference/commandline/plugin_upgrade/) | 更新一个已存在的插件 |
