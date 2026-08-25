---
title: Git笔记
slug: git-notes
date: 2018-02-21 10:55:08
updated: 2020-06-04 12:41:35
categories:
  - git
tags:
  - git
---

# Git学习笔记

## 简介

略

## 安装Git

1. 各系统安装Git
    
    
    - Ubuntu `sudo apt-get install git`
    - Centos `sudo yum install git`
    - Fedora `sudo dnf install git`
    - Max OS X 可在官网下载,或安装’Command Line Tools’,或者通过homebrew安装Git
    - Windows 在官网直接下载安装程序
2. 配置本地名字和邮箱
    
    
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "email@example.com"
    ```

## 创建版本库

1. 初始化一个Git仓库，使用`git init`命令。
2. 添加文件到Git仓库，分两步：
    
    
    - 第一步，使用命令`git add <file>`，注意，可反复多次使用，添加多个文件
    - 第二步，使用命令`git commit -m "xxx"`，完成。

## 时光穿梭机

1. `git status`命令可以让我们时刻掌握仓库当前的状态
2. `git diff`顾名思义就是查看difference,显示的格式正是Unix通用的diff格式

- ### 版本回退
    
    1. 回退`git reset --hard commit_id`
    2. `HEAD`指向的版本就是当前版本,上一个版本`HEAD^`,往上100个版本`HEAD~100`.
    3. `git log`查看提交历史,以便确定要回退到哪个版本.可以加上`--pretty=oneline`参数
    4. 要重返未来，用`git reflog`查看命令历史,以便确定要回到未来的哪个版本.
- ### 工作区和暂存区
    
    廖雪峰-Git工作区和暂存区
    
    
    太多了,懒得写.记不清了看看上面链接的.
- ### 管理修改
    
    每次修改不`add`到暂存区,就不会加入到`commit`
- ### 撤销修改
    
    `git checkout -- file`可以丢弃工作区的修改
    
    
    ```bash
    git checkout -- readme.txt
    ```
    
    
    命令`git checkout -- readme.txt`意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：
    
    
    一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
    
    
    一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
    
    
    总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。
- 小结:
    
    
    - 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。
    - 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。
    - 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
- ### 删除文件
    
    `git rm file`

## 远程仓库

- 略
- 廖雪峰-远程仓库
- 常用远程的命令
    
    
    ```bash
    #关联
    git remote add origin git@server-name:path/repo-name.git
    
    #推送,第一次加上参数 -u
    git push -u origin master
    
    #此后可以使用下面命令推送最新修改
    git push origin master
    
    #从远程库克隆
    git clone URL
    ```

## 分支管理

- ### 创建与合并分支
    
    Git鼓励大量使用分支：
    
    
    查看分支：`git branch`
    
    
    创建分支：`git branch <name>`
    
    
    切换分支：`git checkout <name>`
    
    
    创建+切换分支：`git checkout -b <name>`
    
    
    合并某分支到当前分支：`git merge <name>`
    
    
    删除分支：`git branch -d <name>`
- ### 解决冲突
    
    当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。
    
    
    用`git log --graph`命令可以看到分支合并图
- ### 分支管理策略
    
    合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并
- ### Bug分支
    
    修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；
    
    
    当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场
    恢复的时候，先用`git stash list`查看，然后恢复指定的stash,用命令:
    
    
    ```bash
    git stash apply stash@{0}
    ```
- ### Feature分支
    
    开发一个新feature，最好新建一个分支；
    
    
    如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除
- ### 多人协作
    
    常用
- 查看远程库信息，使用`git remote -v`；
    
    
    - 本地新建的分支如果不推送到远程，对其他人就是不可见的；
    - 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
    - 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
    - 建立本地分支和远程分支的关联，使用`git branch --set-upstream-to=origin/branch-name branch-name`；
    - 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

## 标签管理

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么 时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

- ### 创建标签
    
    常用
    
    
    - 命令`git tag <name>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；
    - `git tag -a <tagname> -m "blablabla..."`可以指定标签信息；
    - `git tag -s <tagname> -m "blablabla..."`可以用PGP签名标签；
    - 命令`git tag`可以查看所有标签
- ### 操作标签
    
    常用
    
    
    - 命令`git push origin <tagname>`可以推送一个本地标签；
    - 命令`git push origin --tags`可以推送全部未推送过的本地标签；
    - 命令`git tag -d <tagname>`可以删除一个本地标签；
    - 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签

## 使用GitHub

略.

## 使用码云

码云(gitee.com)

教程见使用码云-廖雪峰

## 自定义Git

让Git显示颜色，会让命令输出看起来更醒目:

```plain
git config --global color.ui true
```

这样，Git会适当地显示不同的颜色，比如`git status`命令

- ### 忽略特殊文件
    
    忽略某些文件时，需要编写`.gitignore`；
    
    
    `.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理！
    
    
    忽略文件后想要添加该文件,可以用`-f`强制添加到Git:
    
    
    ```plain
    git add -f App.class
    ```
    
    
    
    或者你发现，可能是.gitignore写得有问题，需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查
    
    
    ```plain
    git check-ignore -v App.class
    .gitignore:3:*.class    App.class
    ```
- ### 配置别名
    
    简写命令:
     `st`就表示`status`:
    
    
    ```bash
    #--global参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用
    git config --global alias.st status
    ```
    
    
    
    `co`表示`checkout`，`ci`表示`commit`，`br`表示`branch`:
    
    
    ```bash
    git config --global alias.co checkout
    git config --global alias.ci commit
    git config --global alias.br branch
    ```
    
    
    
    命令`git reset HEAD file`可以把暂存区的修改撤销掉（unstage），重新放回工作区。既然是一个unstage操作，就可以配置一个`unstage`别名:
    
    
    ```bash
    git config --global alias.unstage 'reset HEAD'
    ```
    
    
    配置一个git last，让其显示最后一次提交信息:
    
    
      
    ```bash
    git config --global alias.last 'log -1'
    ```
    
    
    把lg配置成了:
    
    
      
    ```bash
    git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
    ```
    
    
    每个仓库的Git配置文件都放在.git/config文件中.
    而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中.
- ### 搭建Git服务器
    
    1. 安装`git`
        
        
        ```bash
        sudo apt-get install git 或者 sudo yum install git
        ```
    2. 创建一个git用户,用来运行git服务`sudo adduser git`
    3. 创建证书登录:
        收集所有需要登录的用户的公钥，就是他们自己的`id_rsa.pub`文件，把所有公钥导入
        到`/home/git/.ssh/authorized_keys`文件里，一行一个。
    4. 初始化Git厂库:
        
        
        先选定一个目录作为Git仓库，假定是`/srv/sample.git`，在`/srv`目录下输入命令：
        
        
        ```bash
        sudo git init --bare sample.git
        ```
        
        
        
        Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：`sudo chown -R git:git sample.git`
    5. 禁用shell登录:出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件>完成。找到类似下面的一行：
        
        
        ```bash
        git:x:1001:1001:,,,:/home/git:/bin/bash
        ```
        
        
        
        改为：
        
        
        ```bash
        git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
        ```
        
        
        
        这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出
    6. 连接(克隆)远程厂库:
        
        
        现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：
        
        
        ```bash
        git clone git@server:/srv/sample.git
        ```
        
        
        
        Cloning into ‘sample’…
        
        
        warning: You appear to have cloned an empty repository.
        
        
        剩下的推送就简单了。

本笔记为记录学习廖雪峰 Git教程
