---
title: VNC Server安装笔记
slug: vnc-server
date: 2019-02-26 11:23:08
updated: 2020-06-04 12:41:35
tags:
  - linux
  - ubuntu
  - centos
  - vnc server
  - tigervnc
  - vnc4server
---

## Centos 7.2 之 tigervnc-server

1. 远程连接服务器实例.
2. 安装GUI界面.
3. 运行以下命令安装VNC Server.

    ```bash
    yum install tigervnc-server -y
    ```
4. 按以下步骤修改VNC Server配置文件，设置用户名（如本示例中的root）：

    i. 运行命令 `vim /lib/systemd/system/vncserver@.service`

    ii. 按`i`键进入编辑模式

    iii. 将`User=<User>`、`ExecStart`和`PIDFile`的内容替换为以下内容

    ```bash
    User=root
    # Clean any existing files in /tmp/.X11-unix environment
    ExecStartPre=-/usr/bin/vncserver -kill %i
    ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i"
    PIDFile=/root/.vnc/%H%i.pid
    ```
5. 运行以下命令，将 /lib/systemd/system/vncserver@.service 改为 /lib/systemd/system/vncserver@:1.service

    ```bash
    mv /lib/systemd/system/vncserver@.service /lib/systemd/system/vncserver@:1.service
    ```
6. 运行以下命令重启systemd

    ```bash
    systemctl daemon-reload
    ```
7. 运行命令 `vncpasswd`，并按界面提示设置VNC Server连接密码
8. （可选）ECS不允许开启SELinux服务和NetworkManager服务.如果开启了这些服务，运行以下命令修改配置

    ```bash
    vi /etc/selinux/config    # 检查SELinux服务。如果SELINUX对应的值不是disabled，必须改为disabled。
    chkconfig --del NetworkManager    # 关闭NetworkManager服务
    ```
9. 运行以下命令设置开机启动VNC Server

    ```bash
    systemctl enable vncserver@:1.service
    ```
10. 运行以下命令启动VNC Server

    ```bash
    systemctl start vncserver@:1.service
    ```
11. 运行命令 `ps -ef | grep vnc`确认服务是否已经启动。如果返回以下类似结果，说明服务已经启动

    由返回结果可知，服务使用了TCP 5901端口.
12. （可选）如果您的实例上开启了防火墙，需要设置防火墙允许VNC访问

    以firewalld为例，您需要做如下设置

    ```bash
    firewall-cmd --permanent --add-service vnc-server #允许VNC访问
    systemctl restart firewalld.service # 重启firewalld
    ```
13. 登录 ECS管理控制台，在实例所在安全组中 添加安全组规则，放行TCP 5901端口

## Ubuntu 14.04 之 vnc4server

1. 远程连接Linux实例
2. 运行命令 `apt-get update` 更新源
3. 运行以下命令安装vnc4server

    ```bash
    apt-get install vnc4server -y
    ```
4. 运行以下命令开启VNC服务并按界面提示设置连接密码

    ```bash
    vnc4server
    ```

    注意：首次启动会要求设置密码，以后可以使用 `vncpasswd` 修改连接密码。

    如果返回结果里出现类似 `New ':1 (root)' desktop is :1`（代表主机名）的输出，表示 vnc4server 启动成功。程序会自动在当前用户（本示例中为 root）主目录下产生一个 `.vnc` 目录。
5. 运行命令 `ps -ef | grep vnc`确认服务是否已经启动
6. 运行以下命令安装GNOME桌面环境

    ```bash
    apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
    ```
7. （可选）运行以下命令备份原有xstartup文件

    ```bash
    cp ~/.vnc/xstartup  ~/.vnc/xstartup.bak
    ```
8. 按以下步骤修改vnc4server启动文件

    1. 运行以下命令打开文件`vim ~/.vnc/xstartup`
    2. 按`i`键进入编辑模式
    3. 将文件内容替换为以下内容

        ```bash
        #!/bin/sh
        # Uncomment the following two lines for normal desktop:
        # unset SESSION_MANAGER
        # exec /etc/X11/xinit/xinitrc
        [ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
        [ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
        xsetroot -solid grey
        vncconfig -iconic &
        #x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
        #x-window-manager &
        gnome-panel &
        gnome-settings-daemon &
        metacity &
        nautilus &
        gnome-terminal &
        ```
    4. 按`Esc`键退出编辑模式，再输入`:wq`保存并退出
9. 依次运行以下命令生成新的会话

    ```bash
    vncserver -kill :1    #杀掉原来的桌面进程（假设桌面号为:1）
    vncserver :1    # 生成新的会话
    ```
10. 登录 ECS管理控制台，在实例所在安全组中 添加安全组规则，放行TCP 5901端口
