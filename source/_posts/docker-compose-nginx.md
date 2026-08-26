---
title: Docker Compose 部署 Nginx
slug: docker-compose-nginx
date: 2026-08-26 14:45:00
updated: 2026-08-26 14:45:00
categories:
  - docker
tags:
  - docker
  - docker-compose
  - nginx
---
# Docker Compose 部署 Nginx

使用 Docker Compose 部署 Nginx（stable），挂载网站目录、站点配置和日志目录，修改配置后可以热重载，无需重建容器。

## 1. 准备

确认已安装 Docker 和 Docker Compose v2：

```bash
docker --version
docker compose version
```

创建目录结构并进入：

```bash
mkdir -p nginx/{html,conf.d,logs}
cd nginx
```

## 2. docker-compose.yml

创建 `docker-compose.yml`（后续命令默认在 nginx 目录下执行）：

```yaml
services:
  nginx:
    image: nginx:stable-alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./conf.d:/etc/nginx/conf.d:ro
      - ./logs:/var/log/nginx
    healthcheck:
      test: ["CMD-SHELL", "wget -q --spider http://localhost/ || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3
```

## 3. 启动与验证

创建测试页面并启动：

```bash
cd nginx
echo '<h1>Hello Nginx</h1>' > html/index.html
docker compose up -d
docker compose ps
curl http://localhost
```

查看日志：

```bash
docker compose logs -f
tail -f logs/access.log
```

## 4. 自定义站点配置

在 `conf.d` 目录下创建站点配置文件，例如 `default.conf`：

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

修改配置后检查语法并热重载：

```bash
docker exec nginx nginx -t
docker exec nginx nginx -s reload
```

说明：

- `conf.d` 下的配置文件必须以 `.conf` 结尾才会被加载
- `./html` 以只读方式挂载到容器内
- 需要 HTTPS 时，把证书文件挂载到容器（如 `./ssl:/etc/nginx/ssl:ro`），并在站点配置中启用 443

### 文件下载服务器（目录列表）

把文件下载服务器配置写成独立的 `conf.d/download.conf`，与 `default.conf` 互不影响：

```nginx
server {
    listen 80;
    server_name download.local;

    location / {
        root /usr/share/nginx/html/files;
        autoindex on;                # 开启目录列表
        autoindex_exact_size off;    # 显示可读大小（K/M/G）
        autoindex_localtime on;      # 使用本地时间
        charset utf-8;               # 中文文件名不乱码
        #limit_rate 1m;              # 限速下载 1MB
        sendfile on;                 # 开启高效文件传输
    }
}
```

`autoindex` 相关指令说明：

| 指令 | 说明 |
| --- | --- |
| `autoindex on` | 开启目录列表（默认 off） |
| `autoindex_exact_size off` | 显示可读大小，on 时显示精确字节数 |
| `autoindex_localtime on` | 使用本地时间，off 时使用 UTC |
| `charset utf-8` | 避免中文文件名显示乱码 |

放置文件并验证：

```bash
mkdir -p html/files
cp your-file.zip html/files/
docker exec nginx nginx -t
docker exec nginx nginx -s reload
curl -H "Host: download.local" http://localhost/
```

浏览器访问：把 `download.local` 指向服务器 IP（修改本机 hosts），然后打开 `http://download.local/` 即可看到文件列表，点击文件名直接下载。

说明：

- `download.conf` 与 `default.conf` 都监听 80 端口，nginx 根据请求的 Host（`server_name`）区分，两者互不影响
- 本地测试可用 `curl -H "Host: download.local" http://localhost/`，不需要修改 hosts
- 需要独立端口时，在 compose 中增加端口映射（如 `"8080:80"`），并把 `download.conf` 的 `listen` 改为 `8080`
- `./html` 是只读挂载，文件在宿主机 `html/files/` 目录中管理，不需要进入容器

## 5. 常见问题

### 端口被占用

修改宿主机侧端口映射，例如 `"8080:80"`。

### 修改配置后不生效

确认配置文件以 `.conf` 结尾，修改后执行 `docker exec nginx nginx -t` 检查语法，再执行 `nginx -s reload` 热重载。

### 测试页面打不开

检查 `html/index.html` 是否存在，并确认容器状态为 healthy：

```bash
docker compose ps
```

## 参考链接

- Nginx 官方镜像：https://hub.docker.com/_/nginx
- Docker Compose 官方文档：https://docs.docker.com/compose/
