---
title: JAV并发编程艺术-读书笔记
date: 2016-04-01 10:56:43
categories:
- 读书笔记
tags:
- JAVA
- 并发
---

## volatile
对于volatile修饰的变量jvm编译时会增加lock前缀命令，cpu在执行lock前缀命令时会将数据写会内存中（cpu缓存也会更新），并且将将其他cpu缓存相同地址的数据失效
## synchronized
* 普通同步方法，锁当前对象
* 同步静态方法，锁Class
* 同步方法块锁括号中的对象
synchronized使用的锁对象存储在对象的头信息中