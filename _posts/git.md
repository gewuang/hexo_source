---
layout: mypost
title: Git内部原理介绍
date: 2019-12-25 11:56:21
categories: [git]
tags: "git" 
---

# 前言

本文主要是介绍Git内部原理，包含底层命令与文件系统两部分，旨在让我们从了解Git底层原理的运作，从而更好的使用。本文适用于有一定Git基础，并且对Git实现原理比较好奇的同学。

# Git 文件系统简介

首先，先说个概念：

> Git文件系统是**内容寻址文件系统**。

内容寻址文件系统是什么？

简单来说就是一个**键值对数据库**，你可以向数据库中插入任意类型的内容，它会返回一个键值，你可以通过这个键值再次检索插入的内容。

有没有点感觉了，没有也没关系，我们先来分析研究Git文件系统的内容，找到我们想要的答案。

如何得到Git文件系统？

首先，新建个文件夹，如`mkdir mygit`，进入文件夹后使用`git init`命令，初始化Git后得到`.git`文件夹，这个就是我们的文件系统实体，具体操作如下图：

![](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/Git%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F.jpg)

可以看到.git文件夹下已经有很多内容，相关介绍如下图所示：

![](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191223002923.png)

可以看到每个文件或者目录对应的介绍，这里面我们重点关注的主要有**HEAD、objects以及refs**等三个内容，在介绍他们前，我们先了解**底层命令**，用它来打开我们走进Git内部的大门。

# Git底层命令

通过`git help`可以查看高级命令及相关介绍，而如果我们想看底层命令，可以通过命令`git help -g`，下面是一些常用底层命令：

![git底层命令](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/底层命令.png)

了解底层命令可以让我们更直观的理解git的工作原理，比如我们平时操作的`git add & git commit`其实可以用底层命令实现，如下：

![git commit](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/git_commit.png)

上图为一个新建的git仓库，通过使用内部命令`hash-object、update-index、write-tree、commit-tree`等实现了我们的添加和提交的功能，这几部的作用依次是：

1. hash-object 得到test1文件对象的键值
2. updata-index将test1添加到暂存区（实现了add操作）
3. write-tree命令将暂存区内容写入到一个树对象中（这里提到了对象，下面来介绍对象）
4. 最后通过commit-tree命令将树对象提交得到一个提交对象（实现了commit操作）

用底层命令需要四部才能完成我们的工作（其实还有第五步，后面讲文件系统在介绍），而我们平时用的高级命令`add、commit`只需要两步，这么一比确实高级了很多，但通过这四步，我们可以理解下之前我说的Git就是个键值对数据库这个概念是不是理解了很多，Git所有的文件都可以由生成的键值取出，如下图：

```shell
➜ /home/gewuang/profile/mygit git:(master) ✗ > git cat-file -p \
										ce013625030ba8dba906f756967f9e9ca394464a
hello
➜ /home/gewuang/profile/mygit git:(master) ✗ >
```

这个键值`ce013625030ba8dba906f756967f9e9ca394464a`就是我们上面由`hash-object`生成的。

# Git对象

对象是Git文件系统的基础，可分为**树对象、数据对象、提交对象、标签对象**等四种，如下图所示：

![](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/git%E5%AF%B9%E8%B1%A1.jpg)

**数据对象**就是linux系统中的inode节点（详情看上篇[inode介绍](/2019/12/21/inode/)），只存储文件中的内容，数据对象的名字就是它的"inode节点"，也就是索引节点，而文件名等信息则存放在**树对象**中，对比linux系统就是目录文件，**提交对象**当然就是我们的commit_id了，这也是我们平时接触最多的，下图为对象中的内容：

![git object](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225225234.png)

上图主要使用底层命令`cat-file`获取��象信息，首先用`-p`选项打印对象中文件的内容，然后用`-t`打印对象类型。可以看到首先打印的是blob类型，也就是数据对象的内容，然后是树对象内容，最后是提交对象的内容，这里可以看到tree就是我们当时提交时使用的树对象的键值。下面为具体格式介绍：

![git object2](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225230148.png)

这里为什么没有tag对象，因为tag对象就是我们使用`git tag *`时标签的commit id，如下图：

![git tag](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225230643.png)

# Git文件系统原理

介绍完了git底层命令和git对象，我们可以回头再来看Git文件系统原理了。

首先填前面的第一个坑，.git目录下我们关注的三个内容：**HEAD、objects/以及refs/**。

## HEAD

**HEAD**存放的是我们当前的分支信息

```
➜ /home/gewuang/profile/mygit git:(master) > ls
➜ /home/gewuang/profile/mygit git:(master) > cat .git/HEAD
ref: refs/heads/master
```

当我们新建一个分支后，切换当前分支可以发现HEAD内容已经被改变

```
➜ /home/gewuang/profile/mygit git:(master) > git branch branch1
➜ /home/gewuang/profile/mygit git:(master) > git co branch1
Switched to branch 'branch1'
➜ /home/gewuang/profile/mygit git:(branch1) > cat .git/HEAD
ref: refs/heads/branch1
```

当然我们可以直接修改HEAD文件来更改分支，不过这样是不推荐的，你可以骚一点，用底层命令`symbolic-ref`来进行修改，如下：

```
➜ /home/gewuang/profile/mygit git:(branch1) > cat .git/HEAD
ref: refs/heads/branch1
➜ /home/gewuang/profile/mygit git:(branch1) > git symbolic-ref HEAD refs/heads/master
➜ /home/gewuang/profile/mygit git:(master) > cat .git/HEAD
ref: refs/heads/master
```

## objects/

**objects/**目录下存放着我们的数据对象、树对象以及提交对象等等

```shell
➜ /home/gewuang/profile/mygit git:(master) > find .git/objects -type f
.git/objects/2e/4f59e4b1040fca028e89fe687909373463607a
.git/objects/74/9b9d6b710c728b0296098c2f4c3d883566af08
.git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
➜ /home/gewuang/profile/mygit git:(master) >
```

可以看到，我们之前操作的对象，都在这里存着，存放的格式为前两位作为目录名，后续内容作为文件名，这个文件是由zlib压缩过的，我们一般通过`cat-file`命令进行查看。

## refs/

**refs/**目录放置的是引用文件，先收下我对引用的理解：存储指向数据（提交）对象的**指针**。

对，我理解的引用文件就是指针，里面存放的内容就是键值，可以直接找到对应的对象文件，拿到想要的内容，引用的分类和介绍如下图：

![refs](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225232655.png)

HEAD我认为也是一种引用，不过它比较特殊，是**符号引用**，因为它存放的内容为就是文件路径，如master的` refs/heads/master`，像极了软链接的样子。

这里的**refs/head/master**的内容就是我们当前分支的键值了。

```
➜ /home/gewuang/profile/mygit git:(master) > cat .git/refs/heads/master
749b9d6b710c728b0296098c2f4c3d883566af08
➜ /home/gewuang/profile/mygit git:(master) > git gl
749b9d6 [2019-12-25 22:16:45 +0800]  <gewuang> (HEAD -> master, branch1) first commit
➜ /home/gewuang/profile/mygit git:(master) >
```

`remote/`主要是远程引用相关的，这里不多介绍。

## 包文件

git存放数据对象的时候会把整个文件都存到数据对象中，即使有zlib压缩，你可能也会疑惑，我项目这么大，这么多提交，.git目录不是早就爆掉了么，你能想到的设计的人肯定也想到的，所以就有了包文件，他存在的意义就是将对象进行打包，节省空间。

![packet](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225234856.png)

上图介绍的很清楚了（请自动把引用两个字替换成packet，笔误），不过还有一点注意的是相同的文件修改后的数据对象打包后，前一个对象保存的是差异，而文件的全部内容存放在包中此文件最新的数据对象中，因为git认为最新的是人们访问几率最大的，这个设计是为了节省计算的时间，提高效率。

## Git组织结构

讲了这么多，那么git组织结构是什么样的呢？看下图：

![git_fs](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191225234142.png)

文章很长了，不详细介绍了，看不懂就去翻上面，动动脑子去理解才能吸收。

看到这里的都是好孩子，给你个奖励，我自己的一套git别名，用起来会方便很多，如下：

```shell
[alias]
    glf  = log -n 10 --name-only  --format=\"%Cgreen%h %Cred[%ci] %Creset<%an> %Creset %Cgreen%s %Creset
    gl  = log -n 30 --date-order  --format=\"%Cgreen%h %Cred[%ci] %Creset <%an>%C(yellow)%d%Creset %Cres                eset \"
    gll  = log -n 30  --format=\"%Cgreen%H %Cred[%ci] %Creset<%an> %Creset %Cgreen%s %Creset \"
    gl3 = log -n 20  --format=\"%Cgreen%h %Cred[%ci] %Creset<%an> %Creset %Cgreen%s %Creset \" --graph
    gl2 = log --format=\"%Cgreen%h %Cred[%ci] %Creset<%an> %Creset %Cgreen%s %Creset \"
    glc = log --format=\"%Cgreen%h %Cred[%cd] %Creset<committer:%cn> : %Cred[%ad] %Creset<author:%cn> %C
    glc2 = log --format=\"%Cgreen%h %Cred[%ci] %Creset<committer:%cn> : %Cred[%ai] %Creset<author:%cn> %
    glc3 = log --format=\"%Cgreen%h %Cred[%ci] %Creset<committer:%cn> : %Cred[%ai] %Creset<author:%cn> %
            glw = log  -n 20  --format=\"%Cgreen%h %Cred[%ci] %Creset<%an> %Creset %n%Cgreen%s%Creset%n%b  \"
    #| grep \"+0800]\"
    gldetail = log --format=\"%h `[%cd] `<committer:%cn> `[%ad] `<author:%an> ` %s \"
    hist = log --pretty=format:\"%C(yellow)%h %C(red)%d %C(reset)%s %C(green)[%an] %C(blue)%ad\" --topo-order --graph           latest = for-each-ref --sort=-committerdate --format=\"%(committername)@%(refname:short) [%(committerdate:short)] %(contents)\"
        #gldetail = log --format=\"%h [%cd] <committer:%cn> :[%ad] <author:%an>  %s \"

    st = status -uno -s
    st2 = status -s
    co = checkout
    bl = blame --date=short
    #gll = log --format=\"%Cgreen%h %Cred[%ad] %Creset<%an> %Creset %Cgreen%s %Creset \"
    ci = commit
    dt = difftool
    dif = diff --word-diff
        #di = diff --color-words
    di = diff --no-ext-diff
```


