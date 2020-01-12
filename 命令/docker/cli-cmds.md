# Docker 基本命令

这里列出了 Docker 命令行接口的全部命令，但有可能因为宿主机平台不同缺少一些命令。

## 管理命令列表

> 英文描述更容易理解，中文略过

| 命令 | 描述 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker builder](https://docs.docker.com/engine/reference/commandline/builder/) | Manage builds |
| [docker checkpoint](https://docs.docker.com/engine/reference/commandline/checkpoint/) | Manage checkpoints |
| [docker config](https://docs.docker.com/engine/reference/commandline/config/) | Manage Docker configs |
| [docker container](https://docs.docker.com/engine/reference/commandline/container/) | Manage containers |
| [docker context](https://docs.docker.com/engine/reference/commandline/context/) | Manage contexts |
| [docker engine](https://docs.docker.com/engine/reference/commandline/engine/) | Manage the docker engine |
| [docker image](https://docs.docker.com/engine/reference/commandline/image/) | Manage images |
| [docker manifest](https://docs.docker.com/engine/reference/commandline/manifest/) | Manage Docker image manifests and manifest lists |
| [docker network](https://docs.docker.com/engine/reference/commandline/network/) | Manage networks |
| [docker node](https://docs.docker.com/engine/reference/commandline/node/) | Manage Swarm nodes |
| [docker plugin](https://docs.docker.com/engine/reference/commandline/plugin/) | Manage plugins |
| [docker secret](https://docs.docker.com/engine/reference/commandline/secret/) | Manage Docker secrets |
| [docker service](https://docs.docker.com/engine/reference/commandline/service/) | Manage services |
| [docker stack](https://docs.docker.com/engine/reference/commandline/stack/) | Manage Docker stacks |
| [docker swarm](https://docs.docker.com/engine/reference/commandline/swarm/) | Manage Swarm |
| [docker system](https://docs.docker.com/engine/reference/commandline/system/) | Manage Docker |
| [docker trust](https://docs.docker.com/engine/reference/commandline/trust/) | Manage trust on Docker images |
| [docker volume](https://docs.docker.com/engine/reference/commandline/volume/) | Manage volumes |

## 普通命令列表

| 命令 | 描述 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker attach](https://docs.docker.com/engine/reference/commandline/attach/) | 将本地标准输入，输出和错误流附加到正在运行的容器 |
| [docker build](https://docs.docker.com/engine/reference/commandline/build/) | 从 Dockerfile 构建镜像 |
| [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) | 根据容器的更改创建新镜像 |
| [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) | 在容器和本地文件系统之间复制文件/文件夹 |
| [docker create](https://docs.docker.com/engine/reference/commandline/create/) | 创建一个新的容器 |
| [docker deploy](https://docs.docker.com/engine/reference/commandline/deploy/) | 部署新堆栈或更新现有堆栈 |
| [docker diff](https://docs.docker.com/engine/reference/commandline/diff/) | 检查容器文件系统上文件或目录的更改 |
| [docker events](https://docs.docker.com/engine/reference/commandline/events/) | 从服务器获取实时事件 |
| [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) | 在运行状态的容器中执行命令 |
| [docker export](https://docs.docker.com/engine/reference/commandline/export/) | 将容器的文件系统导出为 tar 存档文件 |
| [docker history](https://docs.docker.com/engine/reference/commandline/history/) | 显示镜像的历史记录 |
| [docker images](https://docs.docker.com/engine/reference/commandline/images/) | 镜像列表 |
| [docker import](https://docs.docker.com/engine/reference/commandline/import/) | 从 tarball 文件导入内容以创建文件系统镜像 |
| [docker info](https://docs.docker.com/engine/reference/commandline/info/) | 显示系统范围的信息 |
| [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) | 返回相关 Docker 对象的详细信息 |
| [docker kill](https://docs.docker.com/engine/reference/commandline/kill/) | 终止一个或多个正在运行的容器 |
| [docker load](https://docs.docker.com/engine/reference/commandline/load/) | 从 tar 存档或 STDIN 载入镜像 |
| [docker login](https://docs.docker.com/engine/reference/commandline/login/) | 登录 Docker Registry |
| [docker logout](https://docs.docker.com/engine/reference/commandline/logout/) | 登出 Docker Registry |
| [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) | 提取容器的日志 |
| [docker pause](https://docs.docker.com/engine/reference/commandline/pause/) | 暂停一个或多个容器中的所有进程 |
| [docker port](https://docs.docker.com/engine/reference/commandline/port/) | 列出端口映射或容器的特定映射 |
| [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) | 列出容器 |
| [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) | 从 Registry 中拉取镜像或存储库 |
| [docker push](https://docs.docker.com/engine/reference/commandline/push/) | 将镜像或存储库推送到 Registry |
| [docker rename](https://docs.docker.com/engine/reference/commandline/rename/) | 重命名容器 |
| [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) | 重新启动一个或多个容器 |
| [docker rm](https://docs.docker.com/engine/reference/commandline/rm/) | 删除一个或多个容器 |
| [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) | 删除一个或多个镜像 |
| [docker run](https://docs.docker.com/engine/reference/commandline/run/) | 在新容器中运行命令 |
| [docker save](https://docs.docker.com/engine/reference/commandline/save/) | 将一个或多个镜像保存到 tar 存档（默认情况下流式传输到 STDOUT） |
| [docker search](https://docs.docker.com/engine/reference/commandline/search/) | 在 Docker Hub 中搜索镜像 |
| [docker start](https://docs.docker.com/engine/reference/commandline/start/) | 启动一个或多个已停止的容器 |
| [docker stats](https://docs.docker.com/engine/reference/commandline/stats/) | 显示实时的容器资源使用情况统计流 |
| [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) | 停止一个或多个运行中的容器 |
| [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) | 创建一个引用了 SOURCE_IMAGE 的标签 TARGET_IMAGE |
| [docker top](https://docs.docker.com/engine/reference/commandline/top/) | 显示容器的运行过程 |
| [docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/) | 取消暂停一个或多个容器中的所有进程 |
| [docker update](https://docs.docker.com/engine/reference/commandline/update/) | 动态更新一个或多个容器的配置 |
| [docker version](https://docs.docker.com/engine/reference/commandline/version/) | 显示 Docker 版本信息 |
| [docker wait](https://docs.docker.com/engine/reference/commandline/wait/) | 阻塞直到一个或多个容器停止，然后打印其退出代码 |
