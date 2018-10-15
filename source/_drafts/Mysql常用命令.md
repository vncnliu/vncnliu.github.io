CREATE USER 'root'@'192.168.10.18' IDENTIFIED BY 'root'; 
GRANT ALL ON *.* TO 'root'@'192.168.10.18'; 
mysql --protocol=tcp --host=localhost --user=root --port=3306 --default-character-set=utf8 --comments --database=database < database.dmp

创建存储过程
-- 批量插入数据的sql语句
delimiter $$
DROP PROCEDURE IF EXISTS `sp_insert_atch` $$
CREATE PROCEDURE `sp_insert_atch`(IN number INT)
BEGIN
    DECLARE i INT ;
    SET i = 1;
    #such as 1-2000,2000-4000,
    WHILE i <= number DO 
        IF MOD(i,2000) = 1 THEN 
            SET @sqltext = CONCAT('(''',CONCAT('t',i),''',''',now(),''',',CEIL(10*rand()),')');
        ELSEIF MOD(i,2000) = 0 THEN 
            SET @sqltext = CONCAT(@sqltext,',(''',CONCAT('t',i),''',''',now(),''',',CEIL(10*RAND()),')');
            SET @sqltext = CONCAT('insert into voucher (name,datetime,rank) values',@sqltext);
            prepare stmt FROM @sqltext;
            execute stmt;
            deallocate prepare stmt;
            SET @sqltext = '';
        ELSE 
            SET @sqltext = CONCAT(@sqltext,',(''',CONCAT('t',i),''',''',now(),''',',CEIL(10*RAND()),')');
        END IF;
        SET i = i + 1;
        END WHILE ;
        #process when number is not be moded by 2000
        #such as 2001,4002, 15200,....
        IF @sqltext<>'' THEN 
            SET @sqltext = CONCAT('INSERT INTO voucher (name,datetime,rank) VALUES',@sqltext);
            prepare stmt FROM @sqltext;
            execute stmt;
            deallocate prepare stmt;
            SET @sqltext='';
        END IF; 
END$$

delimiter ;

CREATE TABLE `song`(
    `id` INT NOT NULL AUTO_INCREMENT COMMENT 'Autoincrement element',
    `name` TEXT NOT NULL ,
    `datetime` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `rank` INT NOT NULL ,
    PRIMARY KEY (`id`)
) engine = MyISAM AUTO_INCREMENT = 8102001 DEFAULT CHARSET = gbk;

//查看表大小等信息information_schema
select TABLE_NAME,DATA_LENGTH,INDEX_LENGTH,TABLE_ROWS from information_schema.TABLES WHERE TABLE_SCHEMA='jjh_2016_10' order by TABLE_ROWS desc;
  //查看表描述等
  SHOW FULL COLUMNS FROM information_schema.TABLES;

   show processlist
   
   #### kill
   kill 用于强制关闭死锁查询
   格式如下：
   KILL [CONNECTION | QUERY] thread_id
   show processlist; 查看当前mysql中各个线程状态，找到id即可强制关闭
   
   ####