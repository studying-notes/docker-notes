# ` docker search` 命令详解

> 在 Docker Hub 中搜索镜像

## 基本信息

### 用法

```
docker search [OPTIONS] TERM
```

### 选项

| 名称 | 描述 |
| ---- | ---- |
| `--filter , -f` | 根据提供的条件过滤输出 |
| `--format` | 用给定的 Go 模板格式化输出 |
| `--limit` | 最大搜索结果数量（ `1-100` 默认 `25`） |
| `--no-trunc` | 不截断 `DESCRIPTION` 输出 |

## 示例

### 通过镜像名称搜索

```bash
$ docker search busybox
```

### 通过镜像名称搜索

```bash
$ docker search --no-trunc busybox
```

### 限制搜索结果

```bash
$ docker search --limit 10 busybox
```

### 过滤搜索

`-f, --filter` 参数的格式为 `key=value`，支持指定多个过滤条件，如 `-f "id=cxfans" -f "uid=cute"`。

目前支持的过滤关键词有：

- `stars` (int) - 镜像被关注数量
- `is-automated` (boolean - true or false) - 是否是自动化生成
- `is-official` (boolean - true or false) - 是否是官方镜像

```bash
$ docker search --filter stars=3 busybox
```

```bash
$ docker search --filter is-automated busybox
```

```bash
$ docker search --filter "is-official=true" --filter "stars=3" busybox
```

### 格式化输出

当指定了 `--format`，就会用给定的 Go 模板格式化输出，支持的规则与 Go 的 [text/template](http://golang.org/pkg/text/template/) 标准库一致。

下面列出了Go 模板的有效占位符：

| 占位符 | 描述 |
| :------------- | :------------- |
| `.Name` | Image Name |
| `.Description` | Image description |
| `.StarCount` | Number of stars for the image |
| `.IsOfficial` | “OK” if image is official |
| `.IsAutomated` | “OK” if image build was automated |

当指定 `--format` 选项时，`ps` 命令将完全按照模板声明的方式输出数据，加上 `table` 参数时包含列标题，列项目通过 `:` 分隔。

```bash
$ docker search --format "{{.Name}}: {{.StarCount}}" nginx

nginx: 5441
jwilder/nginx-proxy: 953
richarvey/nginx-php-fpm: 353
million12/nginx-php: 75
webdevops/php-nginx: 70
h3nrik/nginx-ldap: 35
bitnami/nginx: 23
evild/alpine-nginx: 14
million12/nginx: 9
maxexcloo/nginx: 7
```

```bash
$ docker search --format "table {{.Name}}\t{{.IsAutomated}}\t{{.IsOfficial}}" nginx

NAME                                     AUTOMATED           OFFICIAL
nginx                                                        [OK]
jwilder/nginx-proxy                      [OK]
richarvey/nginx-php-fpm                  [OK]
jrcs/letsencrypt-nginx-proxy-companion   [OK]
million12/nginx-php                      [OK]
webdevops/php-nginx                      [OK]
```
