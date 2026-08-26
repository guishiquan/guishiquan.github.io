---
title: Docker Compose 部署 Elasticsearch 7.2 + Kibana
slug: docker-compose-elasticsearch
description: "使用 Docker Compose 部署 Elasticsearch 7.2（单节点）与 Kibana，数据、日志和插件目录均映射到宿主机。"
date: 2026-08-26 14:50:00
updated: 2026-08-26 14:50:00
categories:
  - docker
tags:
  - docker
  - docker-compose
  - elasticsearch
  - kibana
---
# Docker Compose 部署 Elasticsearch 7.2 + Kibana

使用 Docker Compose 部署 Elasticsearch 7.2（单节点）和配套的 Kibana，配置、数据、日志、插件目录都映射到宿主机。Elasticsearch 7.2 是 2019 年的老版本，已停止官方维护，仅建议老项目兼容使用；新项目请使用 8.x/9.x。

## 1. 目录结构与准备

项目目录结构：

```text
├── docker-compose.yml
├── elasticsearch
│   ├── config
│   │   └── elasticsearch.yml
│   ├── data
│   ├── logs
│   └── plugins
└── kibana
    └── kibana.yml
```

确认已安装 Docker 和 Docker Compose v2，然后创建目录结构：

```bash
docker --version
docker compose version

mkdir -p elasticsearch/{config,data,logs,plugins} kibana
```

后续命令默认在项目根目录（`docker-compose.yml` 所在目录）执行。

## 2. 启动前系统配置

1. 调大系统内存映射数（不设置会启动失败）：

   ```bash
   sudo sysctl -w vm.max_map_count=262144
   echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
   ```
2. 设置数据、日志、插件目录权限（容器内以 uid 1000 运行）：

   ```bash
   sudo chown -R 1000:1000 elasticsearch/data elasticsearch/logs elasticsearch/plugins
   ```

## 3. docker-compose.yml

在项目根目录创建 `docker-compose.yml`：

```yaml
services:
  # Elasticsearch 服务
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.1
    container_name: elasticsearch
    restart: unless-stopped
    environment:
      - ES_JAVA_OPTS=-Xms2g -Xmx4g
      - TZ=Asia/Shanghai
      # 开启安全认证时（elasticsearch.yml 中 xpack.security.enabled: true），
      # 通过 ELASTIC_PASSWORD 设置内置用户 elastic 的密码：
      # - ELASTIC_PASSWORD=your-password
      # - xpack.security.enabled=true
    ports:
      - "9200:9200"
    volumes:
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
      - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    healthcheck:
      test: ["CMD-SHELL", "curl -sf http://localhost:9200/_cluster/health || exit 1"]
      # 开启安全认证后，探测命令需要带上账号密码：
      # test: ["CMD-SHELL", "curl -sf -u elastic:your-password http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Kibana 服务
  kibana:
    image: docker.elastic.co/kibana/kibana:7.2.1
    container_name: kibana
    restart: unless-stopped
    ports:
      - "5601:5601"
    environment:
      - TZ=Asia/Shanghai
      # - ELASTICSEARCH_USERNAME=elastic
      # - ELASTICSEARCH_PASSWORD=your-password
    volumes:
      - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
    depends_on:
      elasticsearch:
        condition: service_healthy
```

Kibana 镜像版本必须与 Elasticsearch 一致（本文均为 7.2.1）。

## 4. 配置 Elasticsearch（elasticsearch.yml）

创建 `elasticsearch/config/elasticsearch.yml`：

```yaml
# 集群与节点
cluster.name: es-cluster
node.name: es-node-1

# 网络（http.host 默认继承 network.host，无需重复配置）
network.host: 0.0.0.0
http.port: 9200

# 单节点模式
discovery.type: single-node

# 数据、日志、插件目录（对应挂载的宿主机目录）
path.data: /usr/share/elasticsearch/data
path.logs: /usr/share/elasticsearch/logs
path.plugins: /usr/share/elasticsearch/plugins

# 锁定内存（配合 compose 中的 memlock ulimit）
bootstrap.memory_lock: true

# 关闭安全认证（仅测试环境）
#xpack.security.enabled: false
```

说明：

- 自定义 `elasticsearch.yml` 会替换镜像内的默认配置，`network.host: 0.0.0.0` 必须保留，否则容器外无法访问
- `http.host` 默认继承 `network.host`，设置了 `network.host: 0.0.0.0` 后不需要再写 `http.host`
- `path.data`、`path.logs`、`path.plugins` 指向挂载目录，数据、日志和插件都会持久化到宿主机
- 开启安全认证后，把 compose 中注释掉的 `ELASTIC_PASSWORD` 取消注释即可设置内置用户 `elastic` 的密码；如果该方式不生效，可进入容器执行 `bin/elasticsearch-setup-passwords interactive` 设置。同时 healthcheck 的探测命令要换成带账号密码的版本（见 compose 中注释），否则容器会被判为 unhealthy，Kibana 不会启动
- 修改配置后重启生效：

```bash
docker compose restart elasticsearch
```

## 5. 配置 Kibana（kibana.yml）

创建 `kibana/kibana.yml`：

```yaml
# Kibana 监听地址，0.0.0.0 表示允许容器外访问
server.host: "0.0.0.0"
server.port: 5601

# 连接 Elasticsearch，使用 compose 中的服务名
elasticsearch.hosts: ["http://elasticsearch:9200"]

# 开启安全认证时填写 ES 账号密码（与 compose 中的 ELASTIC_PASSWORD 一致）：
# elasticsearch.username: "elastic"
# elasticsearch.password: "your-password"

# 界面语言
i18n.locale: "zh-CN"
```

`kibana.yml` 已通过 compose 挂载到容器内（`./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:ro`），无需进入容器修改。修改配置后重启生效：

```bash
docker compose restart kibana
```

配置项说明：

| 配置项                                                  | 说明                                                                            |
| ------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `server.host`                                         | 监听地址，`0.0.0.0` 表示允许容器外访问                                        |
| `server.port`                                         | Kibana 端口，默认 5601                                                          |
| `elasticsearch.hosts`                                 | ES 地址列表，容器内使用 compose 服务名`elasticsearch`，不要写成 `localhost` |
| `i18n.locale`                                         | 界面语言，`zh-CN` 为中文                                                      |
| `elasticsearch.username` / `elasticsearch.password` | 开启安全认证时填写 ES 账号密码                                                  |

## 6. 启动与验证

在项目根目录执行：

```bash
docker compose up -d
docker compose ps
curl http://localhost:9200
curl 'http://localhost:9200/_cluster/health?pretty'
curl http://localhost:5601/api/status
```

浏览器访问 `http://localhost:5601` 打开 Kibana 界面。首次启动时 ES 需要一点时间变为 healthy，Kibana 会自动等待，页面打不开就稍等后刷新。

查看日志（默认查看所有服务，也可指定服务）：

```bash
docker compose logs -f
docker compose logs -f kibana
```

ES 自身的错误日志和慢日志会写入宿主机 `elasticsearch/logs` 目录。

## 7. 常见问题

### 启动失败，提示 vm.max_map_count

按第 2 节设置 `vm.max_map_count=262144` 后重新启动。

### 端口被占用

ES 与 Kibana 分别占用 9200 和 5601，修改宿主机侧端口映射即可，例如 `"9201:9200"`、`"5602:5601"`，Kibana 配置里的 `server.port` 保持 5601 不变。

### Kibana 打不开或连不上 Elasticsearch

- 确认 Kibana 与 Elasticsearch 镜像版本一致（本文均为 7.2.1）
- 确认 `kibana.yml` 中 `elasticsearch.hosts` 指向 `http://elasticsearch:9200`
- 首次启动等待 ES 变为 healthy 后刷新页面；查看日志定位问题：

```bash
docker compose logs kibana
```

### 内存不足

示例中 `ES_JAVA_OPTS` 设置为 `-Xms2g -Xmx4g`（初始堆 2g、最大堆 4g），机器内存较小可以调低，反之调高；建议 `-Xms` 与 `-Xmx` 设为相同值，避免运行中动态调整堆大小。

### 生产环境安全

示例关闭了安全认证，仅适合本地或测试环境。生产环境应把 `elasticsearch.yml` 中的 `xpack.security.enabled` 改为 `true`，通过 compose 中的 `ELASTIC_PASSWORD`（取消注释）或 `bin/elasticsearch-setup-passwords` 设置密码，配置证书，并在 `kibana.yml` 中填写 `elasticsearch.username` 和 `elasticsearch.password`。

## 参考链接

- Elasticsearch 官方镜像：https://www.docker.elastic.co/r/elasticsearch
- Kibana 官方镜像：https://www.docker.elastic.co/r/kibana
- Docker Compose 官方文档：https://docs.docker.com/compose/
