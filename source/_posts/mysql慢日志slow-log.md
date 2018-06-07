---
title: mysql慢日志slow-log
date: 2018-06-07 16:34:11
tags: 慢日志 slow-log
---

OS:liunx  mysql：5.7

# 一常用相关参数
```
#查看慢日志相关参数
show variables like '%slow%'；  
show variables like '%log_quer%';
show variables like '%long_quer%'；

#动态设置参数，服务重启作废,建议在my.cnf修改，方便维护
set global log_queries_not_using_indexes=1;
set global innodb_stats_persistent_sample_pages=20;   #默认分析20页
```
# 二如何分析慢日志
分析日志的总方针
```
1 查询次数多且每次查询占用时间长的sql
2 io大的sql 
3 未加索引的  
```
## 1 mysqldumpslow
mysqldumpslow分析报表
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/sql5.png)
```
count：执行次数
time：执行时间
lock：锁定时间
rows：检索出来的行数
root：谁执行的
sql的具体内容
```

## 2 pt-query-digest
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/sql6.png)
```
***总体统计结果
一总共多少sql
overall：多少查询，整合成多少个不一样的查询
二总共执行情况
exec time：执行时间 ——总共  最小  最大
lock time： 锁定时间
rows sent： 推出多少行
rows examine：扫描多少行
query size：
***查询分组统计，对查询进行参数化并分组，按照总执行时长从大到小
三每组query响应情况
responsetime：哪些语句响应时间是最多的
calls：调用多少次
```
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/sql7.png)
四每个sql语句具体分析


# 三定位语句
通过分析慢日志，定位到需要优化的语句，接下来——
explain 查看执行计划