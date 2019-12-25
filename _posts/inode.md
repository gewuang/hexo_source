---
layout: mypost
title: linux文件系统-inode学习整理 
date: 2019-12-21 10:36:21
categories: [linux]
tags: "linux" 
---

# linux文件系统-inode学习整理

## 介绍

linux文件系统可讲的模块有很多，包括文件系统整体架构、文件系统分类、虚拟文件系统以及文件系统存储结构等等，本文主要介绍的是文件系统的存储结构，也就是本文的重点-inode。

## 文件存储结构

首先从开天辟地开始介绍，我们知道数据是保存在磁盘中的，磁盘具体存贮原理细节不在这里进行说明，而磁盘中的存储空间是如何进行管理的？这里就说到了磁盘块的划分：

1. **超级快：**文件系统中第一个块，存放的是文件系统本身的结构信息，包括每个区域的大小以及未被使用的磁盘块等等信息
2. **inode节点表：**超级块的下部分就是inode节点表了，也就是我们上面的inode table。每个inode节点对应一个文件（或目录）的结构，包括了文件的创建时间、权限等信息，下面有详细的介绍。
3. **数据区：**显然它就是用来保存文件内容的区域，这里要介绍下，磁盘上的块大小一样，一般来说为4kb，即连续的八个扇区（512字节），块手是文件存取的最小单位，超过块大小的文件会放到下一个块中。

就像大家知道的，linux一切皆是文件，所以目录项也是文件，不过这个文件中存储的是目录下的文件及子目录组织结构，相应的文件指向了inode的节点，这里需要说明每个文件对应一个inode节点，之后通过inode节点中有关数据区块的信息找到对应的数据。

文件存储结构的整体架构，如下图所示：

![文件存储架构](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215165841.png)


### inode节点

> inode节点详解

inode节点就是文件元数据的存储区，包括了文件如下内容

```
- 文件的字节数
- 文件拥有者的User ID
- 文件的Group ID
- 文件的读、写、执行权限
- 文件的时间戳，共有三个：ctime指inode上一次变动的时间，
       mtime指文件内容上一次变动的时间，atime指文件上一次打开的时间。
- 链接数，即有多少文件名指向这个inode
- 文件数据block的位置
```

可以使用`stat filename` 命令查看：

![inode节点信息](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215214312.png)

基本除了文件内容外的信息都存储在inode节点中。

inode节点的大小一般来说为128或者256个字节，inode节点的总数，在格式化时就给定，一般是每1KB或每2KB就设置一个inode。假定在一块1GB的硬盘中，每个inode节点的大小为128字节，每1KB就设置一个inode，那么inode table的大小就会达到128MB，占整块硬盘的12.8%。

如果想要查看inode大小，可以使用`dump2fs -h /dev/sda1 | grep "Inode size"`查看:

![inode大小](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215215212.png)

如果要查看每个磁盘的inode使用情况，可以使用`df -i`命令查看：

![inode使用情况](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215215108.png)

每个文件都有自己的对应的inode号，这里要说明的是unix/linux系统中主要根据inode号来识别文件，文件名只是我们用来整理和分辨文件的别称，而文件名主要存储在目录项中。

### 目录项

>  目录项的结构

目录项是linux文件系统的重要组成部分，在linux中目录项也是一种文件，不过他内部存储的信息由两部分组成

- 文件名
- inode编号

我们可以通过`ls -ai dirname` 查看目录结构：
![](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215220517.png)

当我们创建目录时，一定会有的两个内容就是`.`和`..`，`.`表示的是当前目录文件所对应的inode号，`..`对应的是当前目录父目录的inode号，其他的就是我们目录下的文件和对应的inode号。

介绍完上面这些信息我们再来看一开始的流程就很清楚了：

> 首先从目录文件中拿到我们所需文件对应的inode号，通过inode号拿到文件的元数据，通过其中所指向的数据块号取出文件内容。

## 创建流程

> 通过创建流程串通知识点

### 文件创建流程

通过前面的内容我们了解到了文件取出的流程，那创建一个文件的流程是什么样的呢？下面我们来介绍下创建文件的流程。

1. **存储inode节点信息：**内核首先找到一块空的inode节点，将文件的信息存在节点中。
2. **存储数据信息：**数据信息即文件信息，内核从未使用的块列表中找到几个数据块（一般是不连续的），如300、230、540等，内核将缓存区中的数据存储到对应的数据块中。
3. **记录分配情况：**存储完信息后，数据块的分配情况记录在inode节点信息中
4. **添加文件名到目录：**最后内核将文件名和对应的inode节点放到目录文件中。

## inode应用扩展

### 硬连接

一般情况下，linux中的文件名和inode号码是一一对应的，不过也可以多个文件名指向同一个inode节点，也就是我们要介绍的**硬链接**。

创建硬链接的命令为`ln 源文件目标文件`，硬链接与正常的文件相同，只是与其他文件共享同一个inode节点，前面介绍的inode节点信息中Links数就是文件名指向的数量，当对其进行删除的时候只会对inode节点中的links数减少1，当为0的时候文件才会真正被删除。

![硬链接](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215224505.png)

这里说明下，目录项中的`.`和`..`也是一种硬链接。

### 软链接

介绍完硬链接，再介绍一种我们平常使用比较多的一种方式：软链接。

`ln -s 源文件 目标文件`是软链接的创建方式，虽然看起来只是多了个选项s，当时内部原理完全不同。

软链接是单独生成一个链接文件，有自己的inode号，是一个单独的文件，这个文件中的信息是链接的文件的信息。

![软链接文件](https://raw.githubusercontent.com/gewuang/cloudimg/master/data/20191215224252.png)

如上图，可以把软链接看做是一个指针，只不过指针里面的内容为所指向文件的路径，这个指针有自己单独的内存空间。

## 参考文章

[理解inode](https://mp.weixin.qq.com/s?__biz=MzI3NzA5MzUxNA==&mid=2664607114&idx=1&sn=a017c12e763285d14a962ac6b73777dc&chksm=f04d806fc73a09792064565050f68c15c996bfe70ffecbc76fa4f3e48b56a64b3c1c5a8c11f8&mpshare=1&scene=1&srcid=1211DeDoAOtew0UTU3RNhmd4&sharer_sharetime=1576391020479&sharer_shareid=9611729a86ebeb77b8f7858bd1081cff&key=f80acc1ad3183d1e3488fe809bb273e8289beb41eb453824dcfdf6728b500953fcdc2259032892602763732453d32b3cffb598877341e596dc68d6be96522be7cdcbf5620c97edd5ad378e82259cb5f5&ascene=1&uin=NzM3NTc5MTA0&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=AauivB%2B7nT2Kzb17AknytBY%3D&pass_ticket=9C2xO5q7uhHGU40DmWaCTcrigOu1V08lxi5UkDVlKNHyveToj50yVLcuABmJLZBm)

[Linux文件系统详解](https://blog.csdn.net/yuexiaxiaoxi27172319/article/details/45241923)
