---
title: 安装Docker
slug: install-docker
date: 2019-01-08 11:14:28
updated: 2020-06-04 12:41:36
categories:
  - docker
tags:
  - docker
  - linux
  - ubuntu
---

# 在Ubuntu上安装Docker

## 准备

### 操作系统要求

需要一个以下版本的64位系统：

- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)
- Ubuntu Xenial 16.04 (LTS)

### 卸载旧版本

旧版Docker称为 `docker`, `docker.io`, 或 `docker-engine`。如果这些软件安装了就卸载掉：

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

`/var/lib/docker/`下存放images， containers，volumes 和 networks将会被保留。新的Docker Engine 叫做 `docker-ce`.

## 安装方法

### 使用repository安装

在一个新机器上第一次安装Docker之前，需要设置Docker repository。

#### 设置repository

1. 更新 `apt` package 的索引 并允许`apt` 安装软件时使用HTTPS:

    ```bash
    $ sudo apt-get update

    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    ```
2. 添加Docker的官方GPG密钥：

    ```bash
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
3. 使用以下命令来设置stable存储库。要添加 edge或test存储库，请在以下命令中的单词后面添加`nightly`或`test`（或同时添加）`stable`。

    ```bash
    $ sudo add-apt-repository \
          "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
          $(lsb_release -cs) \
          stable"
    ```

#### 安装DOCKER引擎

1. 更新`apt`包索引

    ```bash
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
2. 安装最新版本

    ```bash
    $ sudo apt-get install docker-ce
    ```
3. 安装一个特定的Docker CE版本

    1. 列出repo中可用的版本：

        ```bash
        $ apt-cache madison docker-ce
        ```
    2. 按照完全限定包名称安装一个特定版本

        ```bash
        $ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
        ```
4. 通过运行hello-world映像验证是否正确安装了Docker CE。

    ```bash
    $ sudo docker run hello-world
    ```

### 使用软件包安装

如果无法使用Docker的repo来安装，可以下载适用的`.deb`文件并手动安装。

1. 在 `https://download.docker.com/linux/ubuntu/dists/`, 选择你的Ubuntu版本, 在pool/stable/ 并选择amd64，armhf，ppc64el，或者s390x。
2. 安装Docker CE，将下面的`path/to`更改为你下载的Docker软件包的路径。

    ```bash
    $ sudo dpkg -i /path/to/package.deb
    ```
3. 运行hello-world映像验证是否正确安装了Docker CE。

    ```bash
    $ sudo docker run hello-world
    ```

## 后续工作

安装完docker虽然可以使用，但是每次都要加上sudo 才行。

1. 添加docker group

    ```bash
    sudo groupadd docker
    ```
2. 编辑/etc/group，后面是对应的用户名

    ```
    docker:x:…:{USER}
    ```

    或者使用命令添加

    ```bash
    sudo gpasswd -a ${USER} docker
    ```
3. 重启docker服务

    ```bash
    sudo service docker restart
    ```

## 卸载Docker Engine

1. 卸载Docker Engine，CLI 和 Containerd：

    ```bash
    $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
    ```
2. 手动删除所有的images，containers和volumes：

    ```bash
    $ sudo rm -rf /var/lib/docker
    ```
