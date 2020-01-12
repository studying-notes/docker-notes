# `docker top` 命令详解

> 显示容器的运行过程

## 基本信息

### 用法

```
docker top CONTAINER [ps OPTIONS]
```

## 示例

```bash
$ docker top demo
UID                 PID                 PPID                C                   STIME               TTY     
            TIME                CMD
root                30915               30886               0                   Dec29               pts/0   
            00:00:00            bash
```
