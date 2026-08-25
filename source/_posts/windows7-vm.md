---
title: 虚拟机安装Windows7问题
slug: windows7-vm
date: 2019-03-09 07:54:15
updated: 2020-06-04 12:41:36
categories:
  - windows
tags:
  - windows
  - win7
  - 虚拟机
---

# VirtualBox安装Windows7

1. VirtualBox安装虚拟机使用USB2.0/3.0，首先要安装Oracle VM VirtualBox Extension Pack
    
    
    可在此链接中下载
2. 设置->USB设备->USB3.0控制器
3. 由于win7没有usb3.0驱动，本电脑cpu是intel的，因此需要到intel官方网站上下载usb3.0驱动
    
    
    亦可下载一个`usb3.0通用驱动`
4. 在Ubuntu上记得将用户加入vboxusers组，重启系统
    
    
    ```bash
    sudo adduser <username> vboxusers
    ```

# VMware安装Windows7

1. 下载Windows7镜像文件
    
    
    文件名Win7_Ult_SP1_Chinese(Simplified)_x64.iso
    
    
    系统文件缺少usb3.0的驱动
2. 缺少USB3.0的驱动解决办法
    
    
    1. 设置
        
        
        Virtual Machine Settings->USB Controller->USB3.0
        
        
        1. 对VMware的设置完成后，需要给虚拟机里的系统安装USB3.0驱动
        
        
        名称为：usb3.0通用驱动
        
        
        安装时可能无法一次性安装成功需要多次安装。
3. 缺少网卡驱动
    
    
    下载万能网卡通用驱动
