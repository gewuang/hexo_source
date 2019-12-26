---
layout: mypost
title: JAVA基础知识(持续更新) 
date: 2019-12-25 14:06:07
categories: [Java]
tags: "Java" 
---

# 语法基础

# 编程思想

## 类与对象

###Object

概念

> 超父类，所有类的源头，如果不写继承则默认继承Object

方法

> registNatives(): native修饰，
>
> getClass():
>
> hashCode():
>
> equals():
>
> clone():
>
> toString():
>
> notify():
>
> notifyAll():
>
> wait(...):
>
> finalize():

### this

概念

> 当前类的指针（是个对象吧？）

原理

> ?

### super

概念

> 表示父类的引用

原理

> ?

### 抽象

概念

> 类为抽象的概念

分类

> **抽象类：**由abstract修饰的类，不能被实例化
>
> **抽象方法：**用abstract修饰的方法，只能由子类进行实现

### native

概念

> A native method is a Java method whose implementation is provided by non-java code.
>
> 一般为c/c++实现，使用JNI(java native interface)进行通信

使用规则

> 1. native标识符除不能与abstract联用外，可以与其它标识符联用；
> 2. native method方法可以返回任何java类型，包括非基本类型，也可以进行异常控制；
> 3. 如果含有native method方法的类被继承，子类会继承这个native method方法，也可以使用java语言重写 这个方法；
> 4. 如果一个native method方法被fianl标识，它被继承后不能被重写。

实现步骤

> 1. 在Java中声明`native()`方法,然后编译；
>
> 2. 使用javah命令生成.h文件；
>
> 3. 编写.cpp文件实现native导出方法，其中需要包含第二步产生的.h文件（注意其中需包含JDK带的jni.h文件）；
>
> 4. 将本地代码编译成动态库（Windows：*.dll，linux/unix：*.so，mac os x：*.jnilib）；
>
> 5. Java中使用System.loadLibrary()方法加载第四步产生的动态链接库文件，至此，整个过程结束。

## 封装

概念

> 隐藏细节，开放给用户安全的接口

权限访问控制

> **权限关键字：**public、protected、default（不填一样）、private
>
> 权限依次降低

| 作用域    | 当前类 | 同一package普通类 | 其他package普通类 | 同一package子孙类 | 其他package子孙类 |
| --------- | ------ | ----------------- | ----------------- | ----------------- | ----------------- |
| public    | √      | √                 | √                 | √                 | √                 |
| protected | √      | √                 | ×                 | √                 | √                 |
| 默认      | √      | √                 | ×                 | √                 | ×                 |
| private   | √      | ×                 | ×                 | ×                 | ×                 |

原理

> ?

## 继承

概念

> 子类继承父类的属性和方法、减少了重复代码、增加了类结构关系

注意点

> 1. 继承的抽象方法必须实现
> 2. 只能继承父类非private权限的，具体看上面的权限表格

**原理**

> ?

## 多态

概念

> 子类对父类方法的不同实现

条件

> 1. 父类抽象方法
> 2. 多个子类不同实现
> 3. 使用父类类型接收子类对象，并进行调用

原理

> ？
>
> 函数指针指向不同的方法实现

## 接口

概念

> 规范使用，规定一套标准的用法

详解

> 1. 接口所有方法为抽象类
> 2. 接口中的属性为public final类型的（不写也是）
> 3. 接口是制定适用规范，如对虚拟文件系统对外的增删改查标准接口

# Java容器

- [ ] Collection
  - [ ] List
    - [ ] LinkedList
    - [ ] ArrayList
    - [ ] Vector
  - [ ] Stack
  - [ ] Set
    - [ ] HashSet
  - [ ] LinkedHashSet
    - [ ] TreeSet
  - [ ] Queue
    - [ ] PriorityQueue
- [ ] Map
  - [ ] TreeMap
  - [ ] 
  - [ ] HashMap
    - [ ] LinkedHashMap

---------------------------------

如下为java容器分类图（粗体为重点容器）

![java容器分类图](https://images0.cnblogs.com/i/617995/201404/161353012916056.png)

# 异常

# 泛型

# I/O

- [ ] 文件操作

- [ ] 标准输入输出

- [ ] linux IO模式

  > [linux IO模式](https://segmentfault.com/a/1190000003063859)

# 注解与反射

# 图形化界面


