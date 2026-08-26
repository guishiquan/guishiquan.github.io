---
title: Docker安装MySQL
slug: docker-mysql
description: "记录使用 Docker 获取、运行和连接 MySQL 容器，以及修改 root 密码与开放远程访问的基本步骤。"
date: 2019-02-26 12:09:04
updated: 2020-06-04 12:41:35
categories:
  - docker
  - mysql
tags:
  - docker
  - mysql
---

# Docker与MySQL

1. 安装Docker
2. 获得mysql镜像

    - 查看mysql镜像

        `docker search mysql` 可能会连接超时，需要设置加速器。
    - 获得镜像

        `docker pull mysql:<version>`
    - 查看镜像

        `docker images`
    - 使用镜像

        `docker run -p 0.0.0.0:3306:3306 --name first_mysql -it -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7 bash`

        启用一个mysql镜像：`docker run`是启动容器；`i`是交互式操作；`t`是一个终端；`d`是指在后台运行；

        `-P`是指在本地生成一个随机端口用来映射mysql的`3306`端口，`-p 0.0.0.0:3306:3306`将容器的`3306`端口映射到`0.0.0.0:3306`上；`--name first_mysql`容器命名为`first_mysql`;`mysql:<version>`是指要运行的镜像;`bash`是指创建一个交互式shell。
3. 使用镜像

    - 查看已经运行的镜像

        `docker ps -a`
    - 连接到容器

        `docker exec -it second_mysql bash`

        `docker exec`是连接到容器，一个类似于ssh的命令
4. 设置mysql

    - 查看mysql状态

        `service mysql status`
    - 启动mysql

        `service mysql start`
    - 输入`mysql`命令验证是否启动成功

        `show database;`
    - 设置mysql局域网/外部可以连接

        `use mysql;`

        - 修改root用户密码

            `update user set authentication_string = password('root') where user = 'root';`
        - 对root用户授权

            `GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;`
