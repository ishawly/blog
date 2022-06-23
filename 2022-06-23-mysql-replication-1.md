---
title: MySQL 主从复制1
date: 2022-06-23 17:09:00
# description: MySQL 主从复制
categories:
- 数据库
tags:
- mysql
- docker
---

MySQL 主从复制具有读写分离，提升访问性能，备份数据以及故障转移等优点。下面介绍一种最简单的配置方式：两个全新的数据实例，一个作为主库，另一个作为从库。

<!-- more -->

> 此处使用的版本为: MySQL 5.7
>
> 局域网内两台主机（或虚拟机）并运行 docker 服务（方便测试）



0x01 启动数据库实例


```
# 主库 192.168.31.12
docker run --name=mysql_main \
--mount type=bind,src=/home/liber/mysql/etc/my.cnf,dst=/etc/my.cnf \
--mount type=bind,src=/home/liber/mysql/lib,dst=/var/lib/mysql \
--env MYSQL_ROOT_HOST=% \
--env MYSQL_ROOT_PASSWORD=123456 \
--env MYSQL_DATABASE=monday \
-p 3306:3306 \
-p 33060:33060  \
-d mysql/mysql-server:5.7

# 从库 192.168.31.13
docker run --name=mysql_main \
--mount type=bind,src=/home/liber/mysql/etc/my.cnf,dst=/etc/my.cnf \
--mount type=bind,src=/home/liber/mysql/lib,dst=/var/lib/mysql \
--env MYSQL_ROOT_HOST=% \
--env MYSQL_ROOT_PASSWORD=123456 \
--env MYSQL_DATABASE=monday \
-p 3306:3306 \
-p 33060:33060  \
-d mysql/mysql-server:5.7
```

0x02 创建复制账号

```
# 主库上运行
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO repl@'%' IDENTIFIED BY 'passw0rd';
```



0x03 配置主库和备库

```
# 主库 my.cnf

[mysqld]
user=mysql
#log-error=data-dir/host_name.err
#
# replication 
# 推荐指定全路径
log_bin=/var/lib/mysql/mysql-bin
# 服务器 ID 主从唯一
server_id=12


# 备库 my.cnf

[mysqld]
user=mysql
#
# replacation
log_bin          =/var/lib/mysql/mysql-bin
server_id        =13
# 备库中继日志位置，推荐指定全路径
relay_log        =/var/lib/mysql/mysql-relay-log
# 更新记录到日志中
log_slave_updates=1
# 防止普通用户改动数据
read_only        =1
```

0x04 同步数据

```
# 1. 备库指定主库信息
change master to master_host='192.168.31.12',
master_user='repl',
master_password='passw0rd',
master_log_file='mysql-bin.000001',
master_log_pos=0;

# 查看备库状态
show slave status\G
# 显示如下
               Slave_IO_State:
                  Master_Host: 192.168.31.12
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 4
               Relay_Log_File: mysql-relay-log.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: No
            .....
        Seconds_Behind_Master: NULL

# 2. 备库开始同步
start slave;

```

```
# 可以在主备库上运行 show processlist\G  查看主从同步相关进程信息，参考如下：

# 主库
*************************** 3. row ***************************
     Id: 21
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 188
  State: Waiting for master to send event
   Info: NULL
   
   
# 备库   
*************************** 4. row ***************************
     Id: 22
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 188
  State: Slave has read all relay log; waiting for more updates
   Info: NULL
```







参考：

- [2.5.6 Deploying MySQL on Linux with Docker](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-docker.html)
- 高性能MySQL·第三版