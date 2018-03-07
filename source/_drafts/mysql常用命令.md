---
title: mysql常用命令
categories:
- 烂笔头
tags:
- mysql
- 命令
---
## 数据库导入导出
### 1. 导出
如果没有配置环境变量需先进入mysql的bin目录，保证自己有输出sql文件地址的目录有创建文件的权限
mysqldump -u数据库用户名 -p数据库密码 数据库名称 > 输出sql文件地址
### 2. 导入
mysql -u数据库用户名 -p数据库密码 数据库名称 < 输出sql文件地址
### 3. 添加用户
创建用户
CREATE USER 'username'@'host' IDENTIFIED BY 'password'; 
添加权限
GRANT ALL ON *.* TO 'username'@'host';
