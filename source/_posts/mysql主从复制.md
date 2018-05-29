---
title: mysql主从复制
date: 2018-03-12 16:50:28
tags: 主从复制
---
# 一复制的基本原理
```
1主库的二进制日志记录了对DB库的更改
2从库从主库那里获取日志（中继日志relay log）
3在从库中重做relay log，实现data同步
```

# 二复制功能本质——三个线程
```
主库1个线程(B)，从库2个线程(AC)
1从库start slave，从库创建I/O线程A（作用：连接到主库，等待其发送log）
2主库创建线程B发送日志到从库（作用：识别主库中的binlog dump线程，与线程A对接，发送中继日志到从库）
3从库建立线程C（sql线程，读取relay log，并执行日志，实现data同步）
```
# 三常用命令
```
主库：
1检查二进制日志是否开启
show variables like '%log_bin%';
2查看当前主库的日志
show binary logs;
3主库二进制日志状态
show master status;
4显示当前线程
show processlist;
5显示从库
show slave hosts;
从库
1检查当前的复制状态
show slave status;
2从服务器在目录中有2个小文件：
master.info和relay-log.info 。
I/O线程更新master.info（当前读取到的主库日志的位置）
sql线程更新relay-log.info（当前应用到数据库主库的二进制日志的位置，很重要，决定了从库同步的起始点）
```
# 四实战
* 主库master       
* 从库slave
1. 配置文件my.ini设置
* server-id:主从serverid不能相同，建议用ip
* log_bin：二进制日志名字前缀
* master-info和relay-log 采用表形式
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/copy1.png)
2. 创建复制账号
```
CREATE USER 'Replic_user' @'%' identified by '123456'
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON '*' TO 'Replic_user'@'%'
```
3. master主库
```
1 查看master状态，看一下二进制日志是否启动
show master status;
2 flush tables with read lock
* show processlist看一下是否有大事务，防止ftwrl阻塞，导致后续读操作阻塞
3 使用mysqldump导出master数据库文件。以test库为例
4 unlock tables
```
4. slave从库
```
1将主库导出的数据库文件，使用mysql导入到从库
```
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/copy2.png)

5. slave从库实现主从复制

![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/copy3.png)
>master_log_file和mater_log_pos从哪里知道的呢？
* 上面介绍过 sql线程更新relay-log.info：select * from mysql.slave_relay_log_info
6. 验证主从复制是否成功
![image](https://raw.githubusercontent.com/Jaki0615/PIC/master/copy4.png)