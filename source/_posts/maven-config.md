---
title: Maven配置
slug: maven-config
description: "Maven 安装配置：解压安装、设置本地仓库路径并切换阿里云镜像源。"
date: 2020-06-03 16:32:20
updated: 2020-06-04 12:41:35
categories:
  - maven
tags:
  - linux
  - maven
---

下载Maven安装包，配置安装。

# 安装Maven

## 1. 解压

解压下载好的安装包，并拷贝到`/opt`目录下。

## 2. 设置本地仓库

1. 本地新建一个maven仓库文件夹。
2. 修改maven安装目录下的`conf`目录下的`settings`文件，添加：

    ```xml
    <localRepository>/opt/repo</localRepository>
    ```
3. 修改镜像为阿里云

    修改maven安装目录下的`conf`目录下的`settings`文件

    ```xml
    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>central</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
    <!-- 阿里云仓库 -->
    ```

## 3. 环境变量配置

编辑`/etc/profile`或`~/.bashrc`，末尾添加：

```bash
export MAVEN_HOME=/opt/maven
export PATH=$PATH:$MAVEN_HOME/bin
```

重启或执行如下命令后生效：

```bash
source /etc/profile 或 source ~/.bashrc
```
