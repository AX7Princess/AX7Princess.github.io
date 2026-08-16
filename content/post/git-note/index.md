---
description: ""
title: "Git学习笔记"
draft: true
date: "2026-07-03T07:50:31+08:00"
slug: "git note"
categories:
 - null
tags:
 - null
image: ""
---

### 安装Git
[参考文档](https://git-scm.cn/learn)

安装无脑下一步就可以，安装完成后就可以去终端查看是否安装成功
```
git --version
```
当然也有 help和list等常用指令

#### 初次运行Git前的配置

#### 设置身份

安装完 Git 后，首先要做的事情是设置你的用户名和邮件地址。就是告诉Git你是谁。
```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```
更改编辑器，可选，如果喜欢用其他的编辑器的话
```
git config --global core.editor emacs
```
windows系统则需要使用完整编辑器(exe文件)路径地址


### Git基础

#### 获取git
1. 将本地某个文件或文件夹创建为git仓库
2. 从其他地方克隆一个现有的git仓库

**在现有目录初始化**

首先需要cd进入项目目录下，然后输入
```
git init
```
这会创建一个名为 .git 的新子目录，其中包含你所有必要的仓库文件——一个 Git 仓库骨架。

##### 更改默认分支
当你使用 git init 创建新仓库时，Git 会创建一个名为 master 的分支。
git init的作用是在当前文件夹内生成一个.git隐藏文件，里面包含所有版本控制元数据。

将main设置为默认分支
```
git config --global init.defaultBranch main
```
**查看所有配置**
```
git config --list
```

#### 克隆现有仓库
你使用 ```git clone <url>``` 来克隆仓库。

#### 记录仓库中的更改

工作目录中的每个文件要么处于已跟踪状态，要么处于未跟踪状态。已跟踪的文件是指那些被包含在上一次快照中的文件，以及任何新暂存的文件；它们可以是未修改、已修改或已暂存状态。简而言之，已跟踪文件就是 Git 知道的文件。

未跟踪的文件是指除此之外的所有文件——即工作目录中那些既不在上一次快照中，也不在暂存区中的文件。当你首次克隆一个仓库时，所有的文件都将是已跟踪且未修改的，因为 Git 刚刚将它们检出，而你尚未编辑任何内容。

**检查文件状态**

```
git status
```
git add 可以用来开始追踪新文件、暂存文件、以及其他功能。

如```git add README```

####  查看提交历史 

  ```git log```

#### 提交更改

任何尚未暂存的内容——即创建或修改了但自编辑以来尚未运行 git add 的任何文件——都不会进入本次提交。它们将作为已修改的文件保留在你的磁盘上。

```git commit -a "更新说明"
   git commit --amend  撤销
``` 

#### 显示 Git 存储的短名 URL，用于读取和写入该远程设备时：-v
git remote -v
#### 删除文件
#### 别名
```
例：
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
 git config --global alias.st status
```

### 推送和拉取

```git push```将commit提交的文件推动到github等
```git pull```从github等拉取文件