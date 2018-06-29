### 统计微信任务的分布
```sql
select *,CEILING(sum_num/CEILING(((TIME_TO_SEC(end_time)+60*60)-TIME_TO_SEC(begin_time))/60/60)) nums_in_time from
(
SELECT group_id,business_id,wx_account_id,count(*) sum_num,
date_format(time_create,'%Y-%c-%d') time_day ,
(select date_format(time_create,'%H:%i') end_time from voice_robot.t_wx_task where group_id=t.group_id
and business_id=t.business_id and operate=10031 and date_format(time_create,'%Y-%c-%d')=time_day
and action_submit_status=1 and wx_account_id=t.wx_account_id order by time_create desc limit 1) end_time,
(select date_format(time_create,'%H:%i') begin_time from voice_robot.t_wx_task where group_id=t.group_id
and business_id=t.business_id and operate=10031 and date_format(time_create,'%Y-%c-%d')=time_day
and action_submit_status=1 and wx_account_id=t.wx_account_id order by time_create asc limit 1) begin_time
FROM voice_robot.t_wx_task t where operate=10031 and time_create>'20180610' and action_submit_status=1
group by group_id,business_id,wx_account_id,time_day order by group_id,business_id,time_day) t;
```