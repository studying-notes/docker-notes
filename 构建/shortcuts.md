---
date: 2020-08-15T22:20:49+08:00  # 创建日期
author: "Rustle Karl"  # 作者

# 文章
title: "常用 Docker Image 部署步骤合辑"  # 文章标题
url:  "posts/docker/build/shortcuts"  # 设置网页链接，默认使用文件名
tags: [ "docker", "shortcuts"]  # 自定义标签
series: [ "Docker 从入门到放弃"]  # 文章主题/文章系列
categories: [ "学习笔记"]  # 文章分类

# 章节
weight: 20 # 文章在章节中的排序优先级，正序排序
chapter: false  # 将页面设置为章节

index: true  # 文章是否可以被索引
draft: false  # 草稿
---

## Jaeger

```shell
docker run --restart=always -d --name jaeger -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 9411:9411 jaegertracing/all-in-one
```

## Swagger Editor

```shell
docker pull swaggerapi/swagger-editor
```

```shell
docker run --restart=always -d -p 8080:8080 swaggerapi/swagger-editor
```

```shell
localhost:8080
```

## MySQL

```shell
docker pull mysql:latest
```

### 3306 端口

```shell
docker run --restart=always -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql
```

- `-p 3306:3306` ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过宿主机 `ip:3306` 访问到 MySQL 的服务
- `MYSQL_ROOT_PASSWORD=root`：设置 MySQL 服务 root 用户的密码

### 4567 端口

```shell
docker run --restart=always -itd --name mysql -p 4567:3306 -e MYSQL_ROOT_PASSWORD=root mysql
```

## Redis

```shell
docker pull redis
```

D:/OneDrive/相册/Photos

```conf
server {
            listen 12346;
            server_name localhost;
            location ~ /imgs/ {
                root   /usr/share/nginx/html/;
                autoindex on;
            }
            access_log  logs/access.log main;
            error_log  logs/errors.log;
    }
```

```shell
docker run --restart=always --name redis -p 6379:6379 -d redis
```

## Nginx

### Nginx 图片服务器

```shell
docker pull nginx
```

{{<code language="conf" title="nginx.conf" id="1" expand="" collapse="" isCollapsed="false" >}}
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       12346;
        server_name  localhost;
        
        charset utf-8;

        location / {
            root   /usr/share/nginx/html;
            autoindex on;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
{{</code >}}

相册在 D 盘

```shell
docker run --restart=always -d -p 12346:12346 --name nginx -v D:/OneDrive/相册:/usr/share/nginx/html -v D:/OneDrive/Repositories/notes/content/posts/nginx/nginx.conf:/etc/nginx/nginx.conf -v D:/OneDrive/Repositories/notes/content/posts/nginx/logs:/var/log/nginx nginx
```

相册在 H 盘

```shell
docker run --restart=always -d -p 12346:12346 --name nginx -v H:/OneDrive/相册:/usr/share/nginx/html -v H:/OneDrive/Repositories/notes/content/posts/nginx/nginx.conf:/etc/nginx/nginx.conf -v H:/OneDrive/Repositories/notes/content/posts/nginx/logs:/var/log/nginx nginx
```

访问

```shell
http://localhost:12346/
```

## GitLab

Step 0 - 创建互联网络

```shell
docker network create -d bridge gitlab-net
```

Step 1 - 启动 PostgreSQL

```shell
docker run --restart=always --name gitlab-postgresql -d --env 'DB_NAME=gitlabhq_production' --env 'DB_USER=gitlab' --env 'DB_PASS=password' sameersbn/postgresql
```

```shell
docker network connect gitlab-net gitlab-postgresql
```

Step 2 - 启动 Redis

```shell
docker run --restart=always --name gitlab-redis -d sameersbn/redis
```

```shell
docker network connect gitlab-net gitlab-redis 
```

Step 3 - 启动 GitLab

```shell
docker run --name gitlab -d --link gitlab-postgresql:postgresql --link gitlab-redis:redisio --publish 10022:22 --publish 10080:80 --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' sameersbn/gitlab
```

## WordPress

```shell
docker network create -d bridge wp-net
```

```shell
docker run --restart=always --name mariadb --env MYSQL_ROOT_PASSWORD=example -d mariadb
```

```shell
docker run --restart=always --name wordpress --link mariadb:mysql -p 8080:80 -d wordpress
```

```shell

```

```shell

```

```shell

```
