---
title: 安装Docker
slug: install-docker
description: "在 Ubuntu 上通过官方仓库或 deb 包安装 Docker Engine，并配置免 sudo 使用、开机自启、日志驱动和卸载。"
date: 2019-01-08 11:14:28
updated: 2026-09-11 21:32:08
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

## 后续工作（安装后配置）

以下步骤是可选的，用于让 Linux 主机更好地配合 Docker 工作。

### 以非 root 用户管理 Docker

Docker daemon 绑定的是 Unix socket 而不是 TCP 端口，默认只有 root 用户可以访问，其他用户只能通过 `sudo` 使用。daemon 启动时会创建该 socket，`docker` 组的成员即可访问它。

> **注意**：部分 Linux 发行版使用包管理器安装 Docker Engine 时会自动创建 `docker` 组，此时无需手动创建。

在 daemon 仍以 root 运行的前提下，有两种方式免 `sudo` 运行 `docker` 命令：

- 将用户加入 `docker` 组，在整个登录会话中都可以访问
- 按需进入一个受密码保护、启用 Docker 的子 shell

> **警告**：`docker` 组等同于 root 级别的权限，加入前请了解其对系统安全的影响。如果想以非 root 身份运行 Docker，请使用 Rootless 模式。

#### 将用户加入 docker 组

1. 创建 `docker` 组：

    ```bash
    sudo groupadd docker
    ```
2. 将当前用户加入 `docker` 组：

    ```bash
    sudo usermod -aG docker $USER
    ```
3. 注销并重新登录，使组成员关系重新生效；如果在虚拟机中运行 Linux，可能还需要重启虚拟机。也可以执行下面的命令立即激活：

    ```bash
    newgrp docker
    ```
4. 验证不需要 `sudo` 即可运行 `docker`：

    ```bash
    docker run hello-world
    ```

如果之前在加入 `docker` 组前用 `sudo` 运行过 Docker 命令，可能会看到如下错误：

```
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

这说明 `~/.docker/` 目录的权限不正确。可以删除该目录（会自动重建，但自定义配置会丢失），或者修正属主和权限：

```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

#### 按需访问 docker 组（组密码）

组密码是传统的 Unix 访问控制机制，适合单用户工作站。与 `sudo docker` 不同，`newgrp` 让 Docker CLI 仍以你的用户 ID 运行，只是通过 shell 的主组获得 socket 访问权。

1. 确保用户不在 `docker` 组中（组不存在则创建）：

    ```bash
    sudo groupadd --force docker
    sudo gpasswd --delete "$USER" docker
    ```
2. 完全注销桌面或 SSH 会话后重新登录（新开终端不够，组身份仍保留在已有进程的凭据中），确认 `id -nG` 的输出不包含 `docker`。
3. 为 `docker` 组设置密码：

    ```bash
    sudo gpasswd docker
    ```
4. 用 `newgrp` 打开一个以 `docker` 为主组的子 shell，并验证：

    ```bash
    newgrp docker
    id -un   # 仍是你的用户名
    id -gn   # docker
    docker run --rm hello-world
    ```
5. 用完退出子 shell：

    ```bash
    exit
    ```

> **注意**：组密码能减少登录会话中对 Docker socket 的常驻访问，但不会降低授权后获得的 root 级权限，也无法防御以你的用户身份运行的恶意代码，不能当作安全边界。从该子 shell 创建的文件和目录组属主通常是 `docker`，建议只在该 shell 中执行 Docker 相关命令；退出 shell 后，其启动的后台进程仍保留 `docker` 组身份，直到进程退出。
>
> 修改组密码再次执行 `sudo gpasswd docker`；要停用密码验证、恢复为仅组成员可用，执行 `sudo gpasswd --restrict docker`。

### 配置 Docker 开机自启（systemd）

许多现代 Linux 发行版使用 systemd 管理开机启动的服务。Debian 和 Ubuntu 上 Docker 服务默认开机自启，其他使用 systemd 的发行版可以手动启用：

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

要取消开机自启，把 `enable` 换成 `disable`。

还可以通过 systemd unit 文件配置 Docker 服务，例如设置 HTTP 代理、更换 Docker 运行时文件的目录或分区等。

### 配置默认日志驱动

Docker 的默认日志驱动 `json-file` 会把日志以 JSON 格式写入宿主机文件系统。时间一长日志文件会不断增大，可能耗尽磁盘空间。可以选择以下方案之一：

- 为 `json-file` 日志驱动开启日志轮转
- 改用默认就带日志轮转的 `local` 日志驱动
- 使用把日志发送到远程聚合器的日志驱动

## 卸载Docker Engine

1. 卸载Docker Engine，CLI 和 Containerd：

    ```bash
    $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
    ```
2. 手动删除所有的images，containers和volumes：

    ```bash
    $ sudo rm -rf /var/lib/docker
    ```

## 参考链接

- Docker 官方文档（Linux 安装后配置）：https://docs.docker.com/engine/install/linux-postinstall
