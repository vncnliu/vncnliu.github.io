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

### 定时导入微信任务

```sql
-- 查看是否开启定时任务功能
show variables like '%sche%';
set global event_scheduler =1;

-- 创建存储过程
CREATE PROCEDURE add_wx_task_from_manual()
  BEGIN
    declare wx_num int default 0;
    select count(*) into wx_num from t_wx_basic_config where business_id=167 and status=1;
    if(select hour(now())>9 && hour(now())<22)
      then
        start transaction;
        insert into t_wx_task (user_id, group_id, business_id, operate, target_account, message, manual_id, time_create, time_modified)
        select user_id,group_id,business_id,10031,phone as target_account,message,id,now(),now()
        from wx_task_manual
        where status = 0
        order by id
        limit wx_num;
        update wx_task_manual
        set status = 1
        where status = 0
        order by id
        limit wx_num;
        commit;
    end if;
  END;

-- 创建定时任务,会立即执行一次
create event if not exists e_add_wx_task_manual
on schedule every 1 hour
on completion preserve
do call add_wx_task_from_manual();

```