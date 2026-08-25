---
title: MySQL安装与使用
slug: mysql-install-use
date: 2018-02-21 10:55:08
updated: 2020-06-04 12:41:35
tags:
  - mysql
  - ubuntu
categories:
  - mysql
---

## Ubuntu在线安装MySQL

```bash
sudo apt search mysql       //搜索关于mysql的应用
sudo apt-get install mysql-server
sudo apt-get install libmysqlclient20-dev
ln -s  /usr/lib64/mysql/libmysqlclient.so /lib/libmysqlclient.so
```

## 使用MySQL

启动mysql

```bash
service mysqld start      //启动MySql
```

登录mysql

```text
mysql -u root -p            //登录MySql
//若无密码，出现Enter password：可直接登录后设置密码
```

设置密码

```text
mysql> SET PASSWORD FOR `root`@`localhost`=PASSWORD('*****');
```

浏览已经存在的数据库

```text
mysql> show databases;
```

创建数据库test

```text
mysql> CREATE DATABASE test;
mysql> show databases;
```

选中数据库

```text
mysql> use test;
```

浏览已经存在的表

```text
mysql> show tables;
```

新建表

```text
CREATE TABLE demotable(
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `demodata` VARCHAR(255) DEFAULT NULL,
  PRIMARY KEY(`id`)
);
```

查看表中定义

```text
mysql> describe demotable;
```

添加数据

```text
mysql> INSERT INTO demotable(`id`,`demodata`)
    VALUES("1415925320","guiquan");
mysql> INSERT INTO demotable(`demodata`)
    VALUES("test");
mysql> SELECT * FROM demotable;
```

退出MySql客户端

```text
mysql> quit;
```
