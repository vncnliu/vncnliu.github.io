---
title: 一个简单mysql慢查询的改造
date: 2016-03-27 21:07:36
categories:
- 工作日志
tags: 
- mysql
---
最近要在公司的crm系统中加入呼叫中心，在做呼叫记录统计时遇到了问题。
简略的sql与表结构如下
```sql
CREATE TABLE `crm_call_infomation` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `call_no` varchar(255) DEFAULT NULL COMMENT '呼叫号码',
  `user_id_` int(11) DEFAULT NULL COMMENT '发起呼叫用户id',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8;
# 查询语句
SELECT SQL_NO_CACHE count(distinct call_no) as callCustom from crm_call_infomation group by user_id;
```
首先我模拟插入了100万条数据执行一下统计代码竟然要18秒左右,赶紧explain一下。
{% asset_img 1.png explain result %}
看到filesort感觉好像知道了什么，filesort表示mysql不能靠索引完成这次查询的排序，执行的时候看了一下资源管理器io100%了都
大概看了一下问题主要出在count(distinct call_no)上
hhhhhh