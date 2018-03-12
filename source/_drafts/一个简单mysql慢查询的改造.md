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
首先我模拟插入了100万条数据执行一下统计代码竟然要18秒左右,赶紧加个索引：
```sql
ALTER table crm_call_infomation add index user_id_index (`user_id` ASC);
```
果然快了很多
```log
14:51:47	SELECT SQL_NO_CACHE count(distinct call_no) as callCustom,user_id from crm_call_infomation ignore index (user_id_index) group by user_id	100000 row(s) returned	19.657 sec / 0.625 sec

14:52:15	SELECT SQL_NO_CACHE count(distinct call_no) as callCustom,user_id from crm_call_infomation  group by user_id	100000 row(s) returned	0.015 sec / 1.125 sec

```
explain一下看看
```log
# 用索引
explain SELECT count(distinct call_no) as callCustom,user_id from crm_call_infomation group by user_id;
# 输出结果
'1', 'SIMPLE', 'crm_call_infomation', 'index', 'user_id_index', 'user_id_index', '5', NULL, '1098046', NULL
# 手动屏蔽索引
explain SELECT count(distinct call_no) as callCustom,user_id from crm_call_infomation ignore index (user_id_index) group by user_id;
# 输出结果
'1', 'SIMPLE', 'crm_call_infomation', 'ALL', 'user_id_index', NULL, NULL, NULL, '1098046', 'Using filesort'
```
可以看出rows一样不过mysql的文档中也说了对于innodb这个rows只是估计值，主要区别还是在于没有索引的情况下使用了filesort
mysql5.6对filesort的解释[MySQL must do an extra pass to find out how to retrieve the rows in sorted order. The sort is done by going through all rows according to the join type and storing the sort key and pointer to the row for all rows that match the WHERE clause. The keys then are sorted and the rows are retrieved in sorted order.](https://dev.mysql.com/doc/refman/5.6/en/explain-output.html#EXPLAIN Extra Information)
有了索引就会使用索引排序自然就快了（不过貌似并没有减少IO，因为最终还是要读取每条记录以判断call_no是否相同，为什么就快了这么多呢？难道真的就是排序太耗时了？）
通过以下命令可以观察sql执行的详细信息
```sql
# 开启profile
SET @@profiling = 1;
# 查看所有profiles
show profiles;
# 根据ID查询profile的详细信息
show profile for query ${profileId};
# 删除旧的profile信息
SET @@profiling = 0;
SET @@profiling_history_size = 0;
```
比较两个query的profile
```
no index								 use index		
starting	0.000079                     starting	0.000060
checking permissions	0.000004         checking permissions	0.000004
Opening tables	0.000017                 Opening tables	0.000015
init	0.000017                         init	0.000015
System lock	0.000010                     System lock	0.000005
optimizing	0.000003                     optimizing	0.000003
statistics	0.000013                     statistics	0.000016
preparing	0.000103                     preparing	0.000085
Sorting result	0.000003                 Sorting result	0.000238
executing	0.000002                     executing	0.000006
Sending data	0.000005                 Sending data	1.162195
Creating sort index	20.558084            end	0.000030
end	0.000031                             removing tmp table	0.000005
removing tmp table	0.000005             end	0.000003
end	0.000002                             query end	0.000005
query end	0.000008                     closing tables	0.000007
closing tables	0.000009                 freeing items	0.000242
freeing items	0.000302                 cleaning up	0.000009
logging slow query	0.000028
cleaning up	0.000009
```
可以看到关键耗时就在Creating sort index


