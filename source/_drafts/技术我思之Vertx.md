---
title: 技术我思之Vertx
date: 2016-03-27 21:07:36
categories:
- 技术总结
tags: 
- netty
---
java nio
* channel
* buffer
* selector
###### channel
通道和流很类似，不过也有区别
* 通道双向的，流是单向的如FileInputStream，FileOutputStream
* 通道要和buffer绑定使用，只能通过buffer向channle中读写数据
* channel的读写可以异步
###### buffer