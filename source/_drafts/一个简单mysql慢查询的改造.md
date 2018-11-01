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

下面这两个sql第一个20毫秒，第二个要7秒
group 前后数据量没区别，但是第二个貌似进行了全表扫描
```sql
-- sql1
explain select t1.origin_wx_id originWxId,
               t1.target_wx_id targetWxId,
               t1.wx_nickname  wxNickname,
               t1.memo,
               t1.head_picture headPicture,
               t2.message      lastMessage,
               t2.time_create  lastMessageTime,
               t1.type,
               t1.time_create  createTime
        from (select f.origin_wx_id, f.target_wx_id, f.wx_nickname, f.memo, f.head_picture, f.type, f.time_create
              from t_wx_friend f
              where f.origin_wx_id = 'wxid_rg86z6as66io12'
              ) t1
               left join (select *
                          from t_wx_message_task
                          where server_account = 'wxid_rg86z6as66io12'
                            and id in (select max(id)
                                       from t_wx_message_task
                                       where action_submit_status = 1
                                         and action_execution_status = 1
                                       group by server_account, target_account
                                       order by time_create desc)) t2 on t1.target_wx_id = t2.target_account
order by t1.time_create desc limit 100;

-- sql2
explain select t1.origin_wx_id originWxId,
               t1.target_wx_id targetWxId,
               t1.wx_nickname  wxNickname,
               t1.memo,
               t1.head_picture headPicture,
               t2.message      lastMessage,
               t2.time_create  lastMessageTime,
               t1.type,
               t1.time_create  createTime
        from (select f.origin_wx_id, f.target_wx_id, f.wx_nickname, f.memo, f.head_picture, f.type, f.time_create
              from t_wx_friend f
              where f.origin_wx_id = 'wxid_rg86z6as66io12'
              ) t1
               left join (select *
                          from t_wx_message_task
                            where id in (select max(id)
                                       from t_wx_message_task
                                       where action_submit_status = 1
                                         and action_execution_status = 1
                                       and server_account = 'wxid_rg86z6as66io12'
                                       group by server_account, target_account
                                       order by time_create desc)) t2 on t1.target_wx_id = t2.target_account
order by t1.time_create desc limit 100;
```

通过分析执行过程看到sql2使用了临时表，explain也可以看出扫描行数基本和t_wx_message_task表数据总数据量一致。
看来join时id的检索没有生效，导致扫描数过多，而sql1增加了server_account使数据量大大减少，也就不需要临时表了。
通过改写sql2将id的限制换成 in(1,2)同样会全表扫描但是使用如下语句

```sql
-- sql3
explain select t1.origin_wx_id originWxId,
               t1.target_wx_id targetWxId,
               t1.wx_nickname  wxNickname,
               t1.memo,
               t1.head_picture headPicture,
               t2.message      lastMessage,
               t2.time_create  lastMessageTime,
               t1.type,
               t1.time_create  createTime
        from (select f.origin_wx_id, f.target_wx_id, f.wx_nickname, f.memo, f.head_picture, f.type, f.time_create
              from batman_local.t_wx_friend f
              where f.origin_wx_id = 'wxid_rg86z6as66io12'
              ) t1
               left join (select *
                          from batman_local.t_wx_message_task
                            where id in (select mid from ( select max(id) mid
                                       from batman_local.t_wx_message_task
                                       where action_submit_status = 1
                                         and action_execution_status = 1
                                       and server_account = 'wxid_rg86z6as66io12'
                                       group by server_account, target_account
                                       order by time_create desc) tt)) t2 on t1.target_wx_id = t2.target_account
order by t1.time_create desc limit 100;
```
却能过滤join后的数据，可见mysql对子查询和join的优化还是很复杂
sql1 sql2 的profile
```
-- sql1
starting	0.000249
checking permissions	0.000013
checking permissions	0.000003
checking permissions	0.000006
Opening tables	0.000029
init	0.000172
System lock	0.000016
optimizing	0.000021
statistics	0.000150
preparing	0.000043
Sorting result	0.000018
optimizing	0.000014
statistics	0.000085
preparing	0.000037
Sorting result	0.000007
executing	0.000003
Sending data	0.000017
Creating sort index	0.004661
end	0.000007
query end	0.000020
closing tables	0.000014
freeing items	0.000018
removing tmp table	0.000006
freeing items	0.000202
cleaning up	0.000028

-- sql2
starting	0.000237
checking permissions	0.000007
checking permissions	0.000003
checking permissions	0.000006
Opening tables	0.000030
init	0.000171
System lock	0.000015
optimizing	0.000020
statistics	0.000171
preparing	0.000057
Creating tmp table	0.000060
Sorting result	0.000013
optimizing	0.000019
statistics	0.000175
preparing	0.000045
Sorting result	0.000006
executing	0.000004
Sending data	1.028974
executing	0.000004
Sending data	12.632122
Creating sort index	0.000465
end	0.000005
query end	0.000013
removing tmp table	0.000184
query end	0.000008
closing tables	0.000013
freeing items	0.000018
removing tmp table	0.000004
freeing items	0.000120
cleaning up	0.000019

```
explain
{% asset_img sql1.png sql1 %}
{% asset_img sql2.png sql2 %}
{% asset_img sql3.png sql3 %}