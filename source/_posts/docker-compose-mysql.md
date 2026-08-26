---
title: Docker Compose 部署 MySQL（8.4 / 5.7）
slug: docker-compose-mysql
date: 2026-08-26 14:40:00
updated: 2026-08-26 14:40:00
categories:
  - docker
tags:
  - docker
  - docker-compose
  - mysql
---
# Docker Compose 部署 MySQL（8.4 / 5.7）

使用 Docker Compose 部署 MySQL，本文同时给出 MySQL 8.4（LTS，推荐）和 MySQL 5.7（老项目兼容）两种方式，包含数据持久化、自定义配置、初始化脚本和健康检查。每个版本使用独立目录管理，方便单独启停和维护。

## 1. 准备

确认已安装 Docker 和 Docker Compose v2：

```bash
docker --version
docker compose version
```

创建目录结构并进入：

```bash
mkdir -p mysql/{data,conf,init,logs}
cd mysql
```

## 2. 部署 MySQL 8.4（LTS，推荐）

创建 `docker-compose.yml`（后续命令默认在 mysql 目录下执行）：

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: appdb
      MYSQL_USER: app
      MYSQL_PASSWORD: app123
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      - ./data:/var/lib/mysql
      - ./conf:/etc/mysql/conf.d:ro
      - ./init:/docker-entrypoint-initdb.d:ro
      - ./logs:/var/log/mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## 3. 部署 MySQL 5.7（老项目兼容）

创建 mysql57 目录并编写 compose 文件（与 8.4 分开目录）：

```bash
mkdir -p mysql57/{data,conf,init,logs}
cd mysql57
```

创建 `docker-compose.yml`：

```yaml
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: appdb
      MYSQL_USER: app
      MYSQL_PASSWORD: app123
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      - ./data:/var/lib/mysql
      - ./conf:/etc/mysql/conf.d:ro
      - ./init:/docker-entrypoint-initdb.d:ro
      - ./logs:/var/log/mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 5
```

启动方式与 8.4 相同，注意切换到对应目录执行：

```bash
cd mysql57
docker compose up -d
docker compose ps
```

与 8.4 的差异：

- 镜像为 `mysql:5.7`（最后一个版本 5.7.44）
- 5.7 默认认证插件是 `mysql_native_password`，不存在 8.0 的 `caching_sha2_password` 兼容问题
- 5.7 已于 2023 年 10 月停止官方维护，仅建议老项目过渡或本地测试使用
- 官方 5.7 镜像只有 amd64 版本，Apple Silicon 等 arm64 机器无法直接运行

## 4. 常用配置文件 my.cnf

在项目目录下的 `conf` 目录中创建 `my.cnf`（8.4 为 `mysql/conf/my.cnf`，5.7 为 `mysql57/conf/my.cnf`），挂载目录中的 `.cnf` 文件会在容器启动时被 MySQL 自动加载。修改配置后重启容器生效：

```bash
docker compose restart
```

常用配置示例（8.4 和 5.7 通用）：

```ini
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
# 字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# 日志（映射到宿主机 logs 目录）
log-error = /var/log/mysql/error.log
slow_query_log = ON
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2

# 连接数
max_connections = 200
max_allowed_packet = 64M
wait_timeout = 600
interactive_timeout = 600

# 时区
default-time-zone = '+08:00'

# SQL 模式
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

# InnoDB
innodb_buffer_pool_size = 1G
innodb_flush_log_at_trx_commit = 1

# 通用查询日志（可选，生产不建议开启）
# general_log = ON
# general_log_file = /var/log/mysql/general.log

# 二进制日志（可选，文件写入 ./data 目录）
# log-bin = mysql-bin
```

### utf8mb4 与 utf8 的区别

MySQL 中的 `utf8` 实际是 `utf8mb3` 的别名，只支持最多 3 字节的字符；`utf8mb4` 支持完整的 Unicode（最多 4 字节）。两者的主要区别：

| 对比项                     | `utf8`（utf8mb3）              | `utf8mb4`                                    |
| -------------------------- | -------------------------------- | ---------------------------------------------- |
| 单字符最大字节数           | 3 字节                           | 4 字节                                         |
| 字符范围                   | Unicode 基本多文种平面（BMP）    | 完整 Unicode                                   |
| emoji、生僻字等 4 字节字符 | 不支持，写入会报错或变乱码       | 支持                                           |
| 官方态度                   | MySQL 8.0 起已弃用（deprecated） | 推荐使用                                       |
| 常见排序规则               | `utf8_general_ci`              | `utf8mb4_unicode_ci`、`utf8mb4_general_ci` |

因此新部署建议统一使用 `utf8mb4`，本文中的配置也全部使用 `utf8mb4`。代价是单字符最多占用 4 字节，存储占用略大、索引长度上限更小。已经使用 `utf8` 的库表需要迁移时，先备份，再执行：

```sql
ALTER DATABASE 数据库名 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE 表名 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

版本差异：

- 二进制日志保留时间：8.4 使用 `binlog_expire_logs_seconds = 2592000`（单位秒），5.7 使用 `expire_logs_days = 7`（单位天），把对应行加入 `[mysqld]` 即可
- 8.4 如果需要兼容老客户端认证（可选，生产环境不建议），在 `[mysqld]` 中追加：

```ini
[mysqld]
mysql_native_password = ON
authentication_policy = mysql_native_password
```

说明：

- 字符集既可以写在 compose 的 `command` 参数中，也可以写在 `my.cnf` 中，两者等价，取其一即可
- `innodb_buffer_pool_size` 建议设置为机器物理内存的 50% 左右
- `lower_case_table_names = 1` 可以让表名不区分大小写，但必须在初始化数据目录前设置，生产环境谨慎使用

## 5. 启动与使用

启动并验证：

```bash
cd mysql
docker compose up -d
docker compose ps
docker compose logs -f
```

连接 MySQL：

```bash
docker exec -it mysql mysql -uroot -p
```

说明：

- `./data` 持久化数据，删除容器不会丢失
- `./conf` 放自定义配置（如 `my.cnf`）
- `./init` 放首次启动时执行的 `.sql` 初始化脚本（只在数据目录为空时执行）
- `./logs` 映射 MySQL 日志目录，错误日志（`error.log`）和慢查询日志（`slow.log`）会写到宿主机，方便查看和采集
- 健康检查里的 `$$MYSQL_ROOT_PASSWORD` 是 Compose 的转义写法，会取环境变量 `MYSQL_ROOT_PASSWORD` 的值

注意：配置 `log-error` 后，错误日志写入文件，`docker compose logs` 不再显示 MySQL 错误日志，需要直接查看 `logs/error.log`。容器内 MySQL 以 uid 999 运行，如果宿主机 `logs` 目录无法写入，执行：

```bash
sudo chown -R 999:999 logs
```

## 6. 备份与恢复

```bash
# 备份（文件名自动带日期，例如 backup_20260826.sql）
docker exec mysql mysqldump -uroot -p appdb > backup_$(date +%Y%m%d).sql

# 恢复
docker exec -i mysql mysql -uroot -p appdb < backup_20260826.sql
```

## 7. 常见问题

### 端口被占用

修改宿主机侧端口映射，例如 `"3307:3306"`。

### 旧客户端连接失败

MySQL 8.x 默认使用 `caching_sha2_password` 认证，旧版客户端如果连不上，需要升级客户端或为账号改用 `mysql_native_password`（5.7 无此问题）。

### 数据目录权限问题

确认宿主机 `./data` 目录可写；如果使用带 SELinux 的系统（如 CentOS），可能需要为挂载目录添加 `:z` 后缀或调整 SELinux 策略。

### MySQL 5.7 的维护状态

MySQL 5.7 已于 2023 年 10 月停止官方维护（最后一个版本为 5.7.44），存在已知安全风险。仅建议老项目迁移过渡期或本地测试使用，新项目请使用 8.4。

### Apple Silicon 无法运行 mysql:5.7

官方 `mysql:5.7` 镜像只有 amd64 版本。Apple Silicon 等 arm64 机器可以临时加 `platform: linux/amd64` 强制模拟运行，但性能和兼容性没有保证；必要时可考虑使用 MariaDB。

### 同时部署 8.4 和 5.7

使用不同目录，并把 `container_name` 和端口映射改为不同值（例如 `mysql57`、`"3307:3306"`）。两个版本的数据目录不能混用：5.7 的数据目录不能直接给 8.4 使用，8.4 的数据目录也不能回退到 5.7。

### 修改 my.cnf 后配置不生效

确认文件名以 `.cnf` 结尾且放在 `conf` 目录下，修改后执行 `docker compose restart` 重启容器。

## 参考链接

- MySQL 官方镜像：https://hub.docker.com/_/mysql
- Docker Compose 官方文档：https://docs.docker.com/compose/
