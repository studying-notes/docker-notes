# `docker node inspect` 命令详解

> 显示一个或多个节点的详细信息

## 基本信息

### 描述

仅 [**API 1.24**](https://docs.docker.com/engine/api/v1.30/) 及以上版本支持这个命令。

**Swarm** 这个命令适用于 Swarm。

### 用法

```
docker node inspect [OPTIONS] self|NODE [NODE...]
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--format, -f` | 用给定的 Go 模板格式化输出 |
| `--pretty` | 以友好的格式打印信息 |

## 扩展信息

- 默认情况下，该命令将所有结果呈现在 JSON 对象中。

## 示例

### Inspect a node

```bash
$ docker node inspect swarm-manager

[
{
    "ID": "e216jshn25ckzbvmwlnh5jr3g",
    "Version": {
        "Index": 10
    },
    "CreatedAt": "2017-05-16T22:52:44.9910662Z",
    "UpdatedAt": "2017-05-16T22:52:45.230878043Z",
    "Spec": {
        "Role": "manager",
        "Availability": "active"
    },
    "Description": {
        "Hostname": "swarm-manager",
        "Platform": {
            "Architecture": "x86_64",
            "OS": "linux"
        },
        "Resources": {
            "NanoCPUs": 1000000000,
            "MemoryBytes": 1039843328
        },
        "Engine": {
            "EngineVersion": "17.06.0-ce",
            "Plugins": [
                {
                    "Type": "Volume",
                    "Name": "local"
                },
                {
                    "Type": "Network",
                    "Name": "overlay"
                },
                {
                    "Type": "Network",
                    "Name": "null"
                },
                {
                    "Type": "Network",
                    "Name": "host"
                },
                {
                    "Type": "Network",
                    "Name": "bridge"
                },
                {
                    "Type": "Network",
                    "Name": "overlay"
                }
            ]
        },
        "TLSInfo": {
            "TrustRoot": "-----BEGIN CERTIFICATE-----\nMIIBazCCARCgAwIBAgIUOzgqU4tA2q5Yv1HnkzhSIwGyIBswCgYIKoZIzj0EAwIw\nEzERMA8GA1UEAxMIc3dhcm0tY2EwHhcNMTcwNTAyMDAyNDAwWhcNMzcwNDI3MDAy\nNDAwWjATMREwDwYDVQQDEwhzd2FybS1jYTBZMBMGByqGSM49AgEGCCqGSM49AwEH\nA0IABMbiAmET+HZyve35ujrnL2kOLBEQhFDZ5MhxAuYs96n796sFlfxTxC1lM/2g\nAh8DI34pm3JmHgZxeBPKUURJHKWjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMB\nAf8EBTADAQH/MB0GA1UdDgQWBBS3sjTJOcXdkls6WSY2rTx1KIJueTAKBggqhkjO\nPQQDAgNJADBGAiEAoeVWkaXgSUAucQmZ3Yhmx22N/cq1EPBgYHOBZmHt0NkCIQC3\nzONcJ/+WA21OXtb+vcijpUOXtNjyHfcox0N8wsLDqQ==\n-----END CERTIFICATE-----\n",
            "CertIssuerSubject": "MBMxETAPBgNVBAMTCHN3YXJtLWNh",
            "CertIssuerPublicKey": "MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAExuICYRP4dnK97fm6OucvaQ4sERCEUNnkyHEC5iz3qfv3qwWV/FPELWUz/aACHwMjfimbcmYeBnF4E8pRREkcpQ=="
        }
    },
    "Status": {
        "State": "ready",
        "Addr": "168.0.32.137"
    },
    "ManagerStatus": {
        "Leader": true,
        "Reachability": "reachable",
        "Addr": "168.0.32.137:2377"
    }
}
]
```

### Specify an output format

```bash
$ docker node inspect --format '{{ .ManagerStatus.Leader }}' self

false

$ docker node inspect --pretty self
ID:                     e216jshn25ckzbvmwlnh5jr3g
Hostname:               swarm-manager
Joined at:              2017-05-16 22:52:44.9910662 +0000 utc
Status:
 State:                 Ready
 Availability:          Active
 Address:               172.17.0.2
Manager Status:
 Address:               172.17.0.2:2377
 Raft Status:           Reachable
 Leader:                Yes
Platform:
 Operating System:      linux
 Architecture:          x86_64
Resources:
 CPUs:                  4
 Memory:                7.704 GiB
Plugins:
  Network:              overlay, bridge, null, host, overlay
  Volume:               local
Engine Version:         17.06.0-ce
TLS Info:
 TrustRoot:
-----BEGIN CERTIFICATE-----
MIIBazCCARCgAwIBAgIUOzgqU4tA2q5Yv1HnkzhSIwGyIBswCgYIKoZIzj0EAwIw
EzERMA8GA1UEAxMIc3dhcm0tY2EwHhcNMTcwNTAyMDAyNDAwWhcNMzcwNDI3MDAy
NDAwWjATMREwDwYDVQQDEwhzd2FybS1jYTBZMBMGByqGSM49AgEGCCqGSM49AwEH
A0IABMbiAmET+HZyve35ujrnL2kOLBEQhFDZ5MhxAuYs96n796sFlfxTxC1lM/2g
Ah8DI34pm3JmHgZxeBPKUURJHKWjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBS3sjTJOcXdkls6WSY2rTx1KIJueTAKBggqhkjO
PQQDAgNJADBGAiEAoeVWkaXgSUAucQmZ3Yhmx22N/cq1EPBgYHOBZmHt0NkCIQC3
zONcJ/+WA21OXtb+vcijpUOXtNjyHfcox0N8wsLDqQ==
-----END CERTIFICATE-----

 Issuer Public Key:	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAExuICYRP4dnK97fm6OucvaQ4sERCEUNnkyHEC5iz3qfv3qwWV/FPELWUz/aACHwMjfimbcmYeBnF4E8pRREkcpQ==
 Issuer Subject:	MBMxETAPBgNVBAMTCHN3YXJtLWNh
```
