---
date: 2022-07-03T17:50:10+08:00
author: "Rustle Karl"

title: "Prometheus 黑盒监控"
url:  "posts/docker/docs/prometheus/exporter/blackbox"  # 永久链接
tags: [ "Docker", "README" ]  # 标签
categories: [ "Docker 学习笔记" ]  # 分类

toc: true  # 目录
draft: true  # 草稿
---

> 黑盒监控即以用户的身份测试服务的外部可见性，常见的黑盒监控包括 HTTP探针、TCP 探针等用于检测站点或者服务的可访问性，以及访问效率等。

## 什么是黑盒监控？

白盒监控，监控主机的资源用量等运行数据。 这些都是支持业务和服务的基础设施，通过白盒能够了解其内部的实际运行状态，通过对监控指标的观察能够预判可能出现的问题，从而对潜在的不确定因素进行优化。

黑盒监控即以用户的身份测试服务的外部可见性，常见的黑盒监控包括 HTTP 探针、TCP 探针等用于检测站点或者服务的可访问性，以及访问效率等。

## blackbox_exporter 简介

blackbox_exporter 是 Prometheus 官方提供的官方黑盒监控解决方案，其中 exporter 之一，可以提供 http(s)、dns、tcp、icmp 的方式对网络进行探测。

Github地址：

https://github.com/prometheus/blackbox_exporter

目前支持的应用场景：

- ICMP 测试
  - 主机探活机制
- TCP 测试
  - 业务组件端口状态监听
  - 应用层协议定义与监听
- HTTP 测试
  - 定义 Request Header 信息
  - 判断 Http status / Http Respones Header / Http Body 内容
- POST 测试
  - 接口联通性
- SSL 证书过期时间
- 自定义测试（扩展）

## 安装

### docker

```bash
docker pull prom/blackbox-exporter
```

```bash
docker run --rm -p 9115:9115 --name blackbox_exporter -v `pwd`:/config prom/blackbox-exporter --config.file=/config/blackbox.yml
```

## 使用原理

运行 blackbox_exporter 时，需要用户提供探针的配置信息，这些配置信息可能是一些自定义的 http 头信息，也可能是探测时需要的一些 tls 配置，也可能是探针本身的验证行为。在 blackbox_exporter 每一个探针配置称为一个 module，并且以 yaml 配置文件的形式提供给 blackbox_exporter 。每一个 module 主要包含以下配置内容，包括探针类型（prober）、验证访问超时时间（timeout）、以及当前探针的具体配置项：

```bash
  # 探针类型: http https tcp dns icmp
  prober: <prober_string>

  # 超时时间
  [ timeout: <duration> ]  #默认单位秒

  # 探针的详细配置,最多只能配置其中一个
  [ http: <http_probe> ]
  [ tcp: <tcp_probe> ]
  [ dns: <dns_probe> ]
  [ icmp: <icmp_probe> ]
```

`<http_probe>` 可配置参数：

```bash
  # 此探针接受的状态代码。 默认为2xx。
  [ valid_status_codes: <int>, ... | default = 2xx ]

  # 此探针接受的 HTTP 版本。
  [ valid_http_versions: <string>, ... ]

  # 探针将使用的 HTTP 方法。
  [ method: <string> | default = "GET" ]

  # 为探针设置的 HTTP 标头。
  headers:
    [ <string>: <string> ... ]

  # 探针是否将遵循任何重定向。
  [ no_follow_redirects: <boolean> | default = false ]

  # 如果存在SSL，则探测失败。
  [ fail_if_ssl: <boolean> | default = false ]

  # 如果不存在SSL，则探测失败。
  [ fail_if_not_ssl: <boolean> | default = false ]

  # 如果响应主体与正则表达式匹配，则探测失败。
  fail_if_body_matches_regexp:
    [ - <regex>, ... ]

  # 如果响应主体与正则表达式不匹配，则探测失败。
  fail_if_body_not_matches_regexp:
    [ - <regex>, ... ]

  # 如果响应头与正则表达式匹配，则探测失败。 对于具有多个值的标头，如果*至少一个*匹配，则失败。 
  fail_if_header_matches:
    [ - <http_header_match_spec>, ... ]

  # 如果响应头与正则表达式不匹配，则探测失败。 对于具有多个值的标头，如果* none *不匹配，则失败。 
  fail_if_header_not_matches:
    [ - <http_header_match_spec>, ... ]

  # HTTP 探针的 TLS 协议的配置。
  tls_config:
    [ <tls_config> ]

  # 目标的 HTTP 基本身份验证凭据。
  basic_auth:
    [ username: <string> ]
    [ password: <secret> ]
    [ password_file: <filename> ]

  # 目标的承载令牌。
  [ bearer_token: <secret> ]

  # 目标的承载令牌文件。
  [ bearer_token_file: <filename> ]

  # 用于连接到目标的 HTTP 代理服务器。
  [ proxy_url: <string> ]

  # HTTP 探针的 IP 协议（ip4，ip6）
  [ preferred_ip_protocol: <string> | default = "ip6" ]
  [ ip_protocol_fallback: <boolean> | default = true ]

  # 探针中使用的 HTTP 请求的主体。
  body: [ <string> ]
```

## 几种应用场景

### ICMP 测试（主机探活）

可以通过 ping(icmp) 检测服务器的存活,在 `blackbox.yml` 配置文件中配置使用 icmp module：

```yaml
modules:
  icmp:
    prober: icmp
```

在 prometheus 配置文件如下：

```yml
  - job_name: 'blackbox-ping'
    metrics_path: /probe
    params:
      modelus: [icmp]
    static_configs:
    - targets:
      - 172.16.106.208  #被监控端ip
      - 172.16.106.80
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.16.106.84:9115  #blackbox-exporter 所在的机器和端口
```

这里分别配置了名为 blackbox-ping 的采集任务，并且通过 params 指定使用的探针（module）以及探测目标（target）。那问题就来了，假如我们有 N 个目标站点且都需要 M 种探测方式，那么 Prometheus 中将包含 N * M 个采集任务，从配置管理的角度来说显然是不可接受的。 在前面的文章我们介绍了 Prometheus 的 Relabeling 能力，这里我们也可以采用 Relabling 的方式对这些配置进行简化。

这里针对每一个探针服务（如 icmp）定义一个采集任务，并且直接将任务的采集目标定义为我们需要探测的站点。在采集样本数据之前通过 relabel_configs 对采集任务进行动态设置。

- 第1步，根据 `target` 实例的地址，写入 `__param_target` 标签中。`__param_<name>` 形式的标签表示，在采集任务时会在请求目标地址中添加 `<name>` 参数，等同于 `params` 的设置；
- 第2步，获取 `__param_target` 的值，并覆写到 `instance` 标签中；
- 第3步，覆写 `target` 实例的 `__address__` 标签值为 `blackbox_exporter` 实例的访问地址。

通过以上3个 relabel 步骤，即可大大简化 Prometheus 任务配置的复杂度。
![Blackbox Target实例](http://dd-static.jd.com/ddimg/jfs/t1/16811/37/17093/34925/62c16680E2505de78/affc3d0522071e9f.png)

### TCP 测试（监控主机端口存活状态）

在 `blackbox.yml` 配置文件中配置使用 tcp module：

```yaml
modules:
  tcp_connect:
    prober: tcp
```

在 prometheus 配置文件如下:

```yml
  - job_name: 'blackbox-tcp'
    metrics_path: /probe
    params:
      modelus: [tcp_connect]
    static_configs:
    - targets:
      - 172.16.106.208:6443
      - 172.16.106.80:6443
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.16.106.84:9115
```

### HTTP检测（监控网站状态）

http 探针是进行黑盒监控时最常用的探针之一，通过 http 探针能够网站或者 http 服务建立有效的监控，包括其本身的可用性，以及用户体验相关的如响应时间等等。除了能够在服务出现异常的时候及时报警，还能帮助运维同学分析和优化网站体验。

在 `blackbox.yml` 配置文件中配置使用 http module：

```yml
modules:
  http_2xx:
    prober: http
    http:
      method: GET
  http_post_2xx:
    prober: http
    http:
      method: POST
```

在 prometheus 配置文件如下:

```yml
  - job_name: 'blackbox-http'
    metrics_path: /probe
    params:
      modelue: [http_2xx]
    static_configs:
    - targets:
      - http://monitor.mall.demo.com/login
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.16.106.84:9115  #blackbox-exporter 所在的机器和端口
```

通过 prober 配置项指定探针类型。配置项 http 用于自定义探针的探测方式，这里有没对 http 配置项添加任何配置，表示完全使用 http 探针的默认配置，该探针将使用 http get 的方式对目标服务进行探测，并且验证返回状态码是否为 2xx，是则表示验证成功，否则失败。

采集的数据如下：

```bash
# DNS解析时间,单位 s
probe_dns_lookup_time_seconds 0.000199105
# 探测从开始到结束的时间,单位 s,请求这个页面响应时间
probe_duration_seconds 0.010889113
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HTTP 内容响应的长度
probe_http_content_length -1
# 按照阶段统计每阶段的时间
probe_http_duration_seconds{phase="connect"} 0.001083728    #连接时间
probe_http_duration_seconds{phase="processing"} 0.008365885 #处理请求的时间
probe_http_duration_seconds{phase="resolve"} 0.000199105    #响应时间
probe_http_duration_seconds{phase="tls"} 0                  #校验证书的时间
probe_http_duration_seconds{phase="transfer"} 0.000446424   #传输时间
# 重定向的次数
probe_http_redirects 0
# ssl 指示是否将 SSL 用于最终重定向
probe_http_ssl 0
# 返回的状态码
probe_http_status_code 200
# 未压缩的响应主体长度
probe_http_uncompressed_body_length 1766
# http 协议的版本
probe_http_version 1.1
# HELP probe_ip_addr_hash Specifies the hash of IP address. It's useful to detect if the IP address changes.
probe_ip_addr_hash 3.24030434e+09
# 使用的 ip 协议的版本号
probe_ip_protocol 4
# 是否探测成功
probe_success 1
```

### 自定义 HTTP 请求

http 服务通常会以不同的形式对外展现，有些可能就是一些简单的网页，而有些则可能是一些基于 REST 的 API 服务。 对于不同类型的 http 的探测需要管理员能够对 http 探针的行为进行更多的自定义设置，包括：http 请求方法、http 头信息、请求参数等。对于某些启用了安全认证的服务还需要能够对 http 探测设置相应的 auth 支持。对于 https 类型的服务还需要能够对证书进行自定义设置。
如下所示，这里通过 method 定义了探测时使用的请求方法，对于一些需要请求参数的服务，还可以通过 headers 定义相关的请求头信息，使用 body 定义请求内容：

```yaml
http_post_2xx:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{}'
```

如果 http 服务启用了安全认证，blackbox_exporter 内置了对 basic_auth 的支持，可以直接设置相关的认证信息即可：

```yaml
http_basic_auth_example:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Host: "login.example.com"
      basic_auth:
        username: "username"
        password: "mysecret"
```

对于使用了Bear Token 的服务也可以通过 bearer_token 配置项直接指定令牌字符串，或者通过 bearer_token_file 指定令牌文件。
对于一些启用了 https 的服务，但是需要自定义证书的服务，可以通过 tls_config 指定相关的证书信息：

```yml
 http_custom_ca_example:
    prober: http
    http:
      method: GET
      tls_config:
        ca_file: "/certs/my_cert.crt"
```

### 自定义探针行为

在默认情况下 http 探针只会对 http 返回状态码进行校验，如果状态码为 2XX（200 <= StatusCode < 300）则表示探测成功，并且探针返回的指标 probe_success 值为1。
如果用户需要指定 http 返回状态码，或者对 http 版本有特殊要求，如下所示，可以使用 valid_http_versions 和valid_status_codes 进行定义：

```yaml
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: []
```

默认情况下，Blockbox 返回的样本数据中也会包含指标 probe_http_ssl，用于表明当前探针是否使用了 ssl：

```yml
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
```

而如果用户对于 http 服务是否启用 ssl 有强制的标准。则可以使用 fail_if_ssl 和 fail_if_not_ssl 进行配置。fail_if_ssl 为 true 时，表示如果站点启用了 ssl 则探针失败，反之成功。fail_if_not_ssl 刚好相反。

```yaml
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []
      method: GET
      no_follow_redirects: false
      fail_if_ssl: false
      fail_if_not_ssl: false
```

除了基于 http 状态码，http 协议版本以及是否启用 ssl 作为控制探针探测行为成功与否的标准以外，还可以匹配 http 服务的响应内容。使用 fail_if_matches_regexp 和 fail_if_not_matches_regexp 用户可以定义一组正则表达式，用于验证 http 返回内容是否符合或者不符合正则表达式的内容。

```yaml
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      method: GET
      fail_if_matches_regexp:
        - "Could not connect to database"
      fail_if_not_matches_regexp:
        - "Download the latest version here"
```

最后需要提醒的时，默认情况 下http 探针会走 ipv6 的协议。 在大多数情况下，可以使用 preferred_ip_protocol=ip4 强制通过ipv4 的方式进行探测。在 Bloackbox 响应的监控样本中，也会通过指标 probe_ip_protocol，表明当前的协议使用情况：

```yaml
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 6
```

### 检查配置文件

检查配置文件是否书写正确

```bash
cd /data/prometheus
./promtool check config prometheus.yml
```

## 集成 Grafana

在 grafana 导入 blackbox_exporter **9965** 号模板：https://grafana.com/grafana/dashboards/9965

注意：此模板需要安装饼状图插件，下载地址：https://grafana.com/grafana/plugins/grafana-piechart-panel

安装插件，重启grafana生效。

```bash
grafana-cli plugins install grafana-piechart-panel
service grafana-server restart
```

查看数据如下：
![在这里插入图片描述](http://dd-static.jd.com/ddimg/jfs/t1/52954/19/19683/143001/62c16681E16f765e9/0d92133253682bf2.png)

## 小结

黑盒监控相较于白盒监控最大的不同在于黑盒监控是以故障为导向当故障发生时，黑盒监控能快速发现故障，而白盒监控则侧重于主动发现或者预测潜在的问题。一个完善的监控目标是要能够从白盒的角度发现潜在问题，能够在黑盒的角度快速发现已经发生的问题。
