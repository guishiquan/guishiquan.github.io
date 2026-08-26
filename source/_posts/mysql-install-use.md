---
title: MySQL安装与使用
slug: mysql-install-use
description: "Ubuntu 在线安装 MySQL，以及登录、建库建表、增删改查等基本使用命令。"
date: 2018-02-21 10:55:08
updated: 2020-06-04 12:41:35
categories:
  - mysql
tags:
  - mysql
  - ubuntu
---

# MySQL 安装与使用

## Ubuntu在线安装MySQL

```bash
sudo apt search mysql   # 搜索关于mysql的应用
```

```bash
sudo apt-get install mysql-server
```

```bash
sudo apt-get install libmysqlclient20-dev
```

```bash
ln -s /usr/lib64/mysql/libmysqlclient.so /lib/libmysqlclient.so
```

## 使用MySQL

启动mysql

```bash
service mysqld start   # 启动MySql
```

登录mysql

```bash
mysql -u root -p   # 登录MySql
# 若无密码，出现Enter password：可直接登录后设置密码
```

设置密码

```sql
mysql> SET PASSWORD FOR `root`@`localhost` = PASSWORD(`*****`);
```

浏览已经存在的数据库

```sql
mysql> show databases;
```

创建数据库test

```sql
mysql> CREATE DATABASE test;
mysql> show databases;
```

选中数据库

```sql
mysql> use test;
```

浏览已经存在的表

```sql
mysql> show tables;
```

新建表

```sql
CREATE TABLE demotable(
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `demodata` VARCHAR(255) DEFAULT NULL,
  PRIMARY KEY(`id`)
);
```

查看表中定义

```sql
mysql> describe demotable;
```

添加数据

```sql
mysql> INSERT INTO demotable(`id`,`demodata`)
    VALUES("1415925320","guiquan");
mysql> INSERT INTO demotable(`demodata`)
    VALUES("test");
mysql> SELECT * FROM demotable;
```

退出MySql客户端

```sql
mysql> quit;
```
