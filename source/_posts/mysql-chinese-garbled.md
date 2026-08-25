---
title: MySQL中文乱码
slug: mysql-chinese-garbled
date: 2018-02-21 10:55:08
updated: 2020-06-04 12:41:35
categories:
  - mysql
tags:
  - mysql
  - 中文乱码
---

修改`mysql.cnf`文件，一般存放于`/etc/mysql/conf.d/mysql.cnf`

```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```
