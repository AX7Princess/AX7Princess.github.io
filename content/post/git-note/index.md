---
description: ""
title: "Git学习笔记"
draft: true
date: "2026-07-03T07:50:31+08:00"
slug: "git note"
categories:
 - 
tags:
 - 
image: ""
---

### 安装Git
[参考文档](https://git-scm.cn/learn)

安装Git
Linux： 
```
sudo apt install git-all
```
Windows：浏览器搜索git去官网，或者使用国内镜像
[链接](https://git-scm.cn/install/windows)

安装无脑下一步就可以，安装完成后就可以去终端查看是否安装成功
```
git --version
```
当然也有 help和list等常用指令

#### 初次运行Git前的配置
*这里实际遇到问题都是找AI，实际没接触到，日后可能会使用到还是记录一下*

Git 自带一个名为 git config 的工具，它允许你获取和设置配置变量，从而控制 Git 的外观和操作。这些变量存储在三个不同的位置：
1. [path]/etc/gitconfig 文件：包含系统上每个用户及他们所有仓库的通用值。如果你在运行 git config 时加上 --system 选项，Git 就会读写该文件。由于这是系统级配置文件，因此需要管理员或超级用户权限才能修改。

2. ~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户生效。你可以传递 --global 选项，让 Git 读写此文件，这会影响你系统上运行的所有仓库。

3. 当前仓库的 Git 目录中的 config 文件（即 .git/config）：仅针对当前仓库有效。你可以使用 --local 选项强制 Git 读写此文件，不过这也是默认行为。当然，你需要进入某个 Git 仓库中才能正确使用该选项。

可以通过以下命令查看所有的配置以及它们所在的配置文件：
```
git config --list --show-origin
```
#### 你的身份

安装完 Git 后，首先要做的事情是设置你的用户名和邮件地址。就是告诉Git你是谁。
```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```
更改编辑器
```
git config --global core.editor emacs
```
windows系统则需要使用完整编辑器(exe文件)路径地址


### Git基础

#### 获取git
1. 将一个当前未进行版本控制的本地目录转换为git仓库
2. 从其他地方克隆一个现有的git仓库

**在现有目录初始化**

首先需要cd进入项目目录下，然后输入
```
git init
```
这会创建一个名为 .git 的新子目录，其中包含你所有必要的仓库文件——一个 Git 仓库骨架。

##### 更改默认分支
当你使用 git init 创建新仓库时，Git 会创建一个名为 master 的分支。从 Git 2.28 版本开始，你可以为初始分支设置不同的名称。

git init的作用是在当前文件夹内生成一个.git隐藏文件，里面包含所有版本控制元数据。

*大树的枝干（什么是分支，这是ai的解释，类似于Word的历史版本，但要强大的多）*

*主干（Main）：是这棵树的命脉，必须保持绝对稳定。*

*侧枝（Feature）：是从主干上长出来的新枝条。它可以在外面自由生长（添新叶子=写新代码），哪怕被风吹断了（代码写崩了），也不会让整棵树枯死。*

*嫁接（Merge）：当这根侧枝长结实了（测试通过），园丁就把它嫁接回主干上，让它成为主干的一部分。*

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
没有任何已跟踪的文件被修改则还会显示
```
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
```
假设你向项目中添加了一个新文件，一个简单的 README 文件。如果该文件之前不存在，且你运行 git status，你将看到如下未跟踪的文件
```
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```
Untracked files未跟踪基本上意味着 Git 发现了一个你在上一个快照（提交）中没有的文件，且尚未被暂存；在你明确告诉 Git 之前，它不会将该文件包含在你的提交快照中为了开始跟踪一个新文件，你需要使用 ```git add <目录或文件名>``` 命令跟踪文件，如果是目录则会递归的添加目录下所有文件

git add 可以用来开始追踪新文件、暂存文件、以及其他功能。

如```git add README```

再次查看状态会显示
```
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)

    new file:   README
```
此时会变成暂存状态，接下来进行提交会将文件写进后续的历史快照里，如果你在运行 git add 后修改了文件，则必须再次运行 git add 以暂存该文件的最新版本。
#### 提交更改

任何尚未暂存的内容——即创建或修改了但自编辑以来尚未运行 git add 的任何文件——都不会进入本次提交。它们将作为已修改的文件保留在你的磁盘上。

```git commit```

#### 删除文件

要从 Git 中删除文件，必须从已跟踪的文件中将其删除（更准确地说是从暂存区中删除），然后提交。```git rm``` 命令可以做到这一点，它还会从你的工作目录中删除该文件，这样你在下次运行时就不会看到它作为未跟踪文件出现。

如果你只是从工作目录中删除文件，它会出现在 git status 输出的“Changes not staged for commit”（即未暂存）区域下,然后，如果你运行 git rm，它会执行暂存文件的删除操作
```
git rm PROJECTS.md
  ...
 Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md

```
#### 撤销提交

请务必小心，因为有些撤销操作是不可逆的。这是 Git 中少数几个如果操作不当可能会丢失数据的地方。此命令会将暂存区中的内容提交。如果你自上次提交后没有做任何修改（例如，在上一次提交后立即运行此命令），那么快照将完全保持不变，你唯一能改变的就是提交信息。

```git commit --amend```

### 推送和拉取

```git push```将commit提交的文件推动到github等
```git pull```从github等拉取文件