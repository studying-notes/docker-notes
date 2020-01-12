# `docker context inspect` 命令

> 显示一个或多个上下文的详细信息

## 基本信息

### 用法

```
docker context inspect [OPTIONS] [CONTEXT] [CONTEXT...]
```

### 选项

| 名称 | 描述 |
| ------- | ------- |
| `--format , -f` | 用给定的 Go 模板格式化输出 |

## 示例

```bash
$ docker context inspect "local+aks"

[
    {
        "Name": "local+aks",
        "Metadata": {
            "Description": "Local Docker Engine + Azure AKS endpoint",
            "StackOrchestrator": "kubernetes"
        },
        "Endpoints": {
            "docker": {
                "Host": "npipe:////./pipe/docker_engine",
                "SkipTLSVerify": false
            },
            "kubernetes": {
                "Host": "https://simon-aks-***.hcp.uksouth.azmk8s.io:443",
                "SkipTLSVerify": false,
                "DefaultNamespace": "default"
            }
        },
        "TLSMaterial": {
            "kubernetes": [
                "ca.pem",
                "cert.pem",
                "key.pem"
            ]
        },
        "Storage": {
            "MetadataPath": "C:\\Users\\simon\\.docker\\contexts\\meta\\cb6d08c0a1bfa5fe6f012e61a442788c00bed93f509141daff05f620fc54ddee",
            "TLSPath": "C:\\Users\\simon\\.docker\\contexts\\tls\\cb6d08c0a1bfa5fe6f012e61a442788c00bed93f509141daff05f620fc54ddee"
        }
    }
]
```
