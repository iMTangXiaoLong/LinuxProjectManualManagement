# MySQL主从集群部署

## 一、节点部署

### 1.准备环境

> MySQL下载地址：https://downloads.mysql.com/archives/community/
>
> MySQL版本（二进制包）：https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.43-linux-glibc2.12-x86_64.tar.gz



### 2.前提准备

> 卸载自带mariadb数据库
>
> [root@MYSQL-MASTER ~]# rpm -qa|grep mariadb
>
> [root@MYSQL-MASTER ~]# rpm -e mariadb-libs-5.5.68-1.el7.x86_64 --nodeps



### 3.创建用户、用户组及目录、赋权

```shell
[root@MYSQL-Master ~]# groupadd mysql
[root@MYSQL-Master ~]# useradd -d /home/mysql -g mysql -m mysql
[root@MYSQL-Master ~]# mkdir software
[root@MYSQL-Master ~]# mkdir -p /home/mysql/software
[root@MYSQL-Master ~]# mkdir -p /home/mysql/yunwei
[root@MYSQL-Master ~]# chown -R mysql:mysql /home/mysql
[root@MYSQL-Master ~]# mkdir -p /data/mysql/data
[root@MYSQL-Master ~]# chown -R mysql:mysql /data/mysql
```



### 4.上传并解压MySQL安装包

```shell
# FTP上传mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz
[root@MYSQL-Master ~]# tar zxf $HOME/software/mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz -C /usr/local
[root@MYSQL-Master ~]# mv /usr/local/mysql-5.7.44-linux-glibc2.12-x86_64 /usr/local/mysql-5.7.44
[root@MYSQL-MASTER ~]# rpm -e mariadb-libs-5.5.68-1.el7.x86_64 --nodeps
[root@MYSQL-Master ~]# touch /etc/my.cnf
```



### 5.编辑配置文件

```shell
[root@MYSQL-Master ~]# vi /etc/my.cnf
[mysqld]
# MySQL端口设置
port=3306

# MySQL安装路径
basedir=/usr/local/mysql-5.7.44

# MySQL数据存放路径
datadir=/data/mysql/data

# MySQL字节套sock存放路径
socket=/tmp/mysql.sock
socket=/data/mysql/data/mysql.sock

# MySQL日志存放
log-error=/data/mysql/mysqld.log

# MySQL服务PID存放路径
pid-file=/data/mysql/mysqld.pid

# MySQL字符集设置
character_set_server=utf8mb4

# MySQL最大连接数设置
max_connections=64

# innodb设置
innodb_buffer_pool_size = 256M
innodb_log_file_size = 128M
innodb_flush_method = O_DIRECT
innodb_flush_log_at_trx_commit = 0

# MySQL客户端设置
[client]
default-character-set=utf8mb4
socket=/data/mysql/data/mysql.sock
```



### 6.初始化数据库

```shell
[root@MYSQL-Master ~]# chown -R mysql:mysql /usr/local/mysql-5.7.44
[root@MYSQL-Master ~]# cd /usr/local/mysql-5.7.44/bin
[root@MYSQL-Master bin]# ./mysqld --defaults-file=/etc/my.cnf --user=mysql --initialize
[root@MYSQL-Master bin]# grep 'temporary password' /data/mysql/mysqld.log 
2023-11-09T05:51:34.353527Z 1 [Note] A temporary password is generated for root@localhost: ************
```



### 7.配置MySQL启动项

```shell
[root@MYSQL-Master ~]# cp /usr/local/mysql-5.7.44/support-files/mysql.server /etc/init.d/mysql
[root@MYSQL-Master ~]# chkconfig mysql on
[root@MYSQL-Master ~]# systemctl start mysql
[root@MYSQL-Master ~]# systemctl status mysql
● mysql.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/rc.d/init.d/mysql; bad; vendor preset: disabled)
   Active: active (exited) since Thu 2023-11-09 10:21:54 CST; 4s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 9791 ExecStart=/etc/rc.d/init.d/mysql start (code=exited, status=0/SUCCESS)

Nov 09 10:21:54 MYSQL-Master systemd[1]: Starting LSB: start and stop MySQL...
Nov 09 10:21:54 MYSQL-Master mysql[9791]: /etc/rc.d/init.d/mysql: line 239: my_print_defaults: command not found
Nov 09 10:21:54 MYSQL-Master mysql[9791]: /etc/rc.d/init.d/mysql: line 259: cd: /usr/local/mysql: No such file or directory
Nov 09 10:21:54 MYSQL-Master mysql[9791]: Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
Nov 09 10:21:54 MYSQL-Master systemd[1]: Started LSB: start and stop MySQL.
```

#### 注：其他报错

```shell
报错1：
# 以上报错需要修改/etc/rc.d/init.d/mysql文件或者是/usr/local/mysql-5.7.44/support-files/mysql.server（尽量修改mysql文件）
[root@MYSQL-Master ~]# vi /etc/rc.d/init.d/mysql
basedir=/usr/local/mysql-5.7.44/
datadir=/data/mysql/data
# 保存退出即可
报错2：
[root@MYSQL-Master ~]# systemctl start mysql
Warning: mysql.service changed on disk. Run 'systemctl daemon-reload' to reload units.
# 报错是因为修改了配置以后需要再次执行 'systemctl daemon-reload' 该命令即可，重新刷新一下，再执行即可
```



### 8.创建root和mysql环境变量

```shell
[root@MYSQL-MASTER ~]# vi $HOME/.bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/usr/local/mysql-5.7.44/bin

export PATH

[root@MYSQL-MASTER ~]# source $HOME/.bash_profile
```



### 9.修改MySQL最高权限root密码

```shell
[root@MYSQL-MASTER bin]# mysql -uroot -p'!Ikhhjl_+5bB'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.44

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'test1234';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```



### 10.创建MySQL普通用户

```shell
[root@MYSQL-MASTER bin]# mysql -uroot -p'test1234'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.44 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database yunwei_auto character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql> grant all privileges on yunwei_auto.* to yunwei_auto@localhost identified by 'test1234';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```



### 11.MySQL服务启停命令

```shell
# 启动
[root@MYSQL-MASTER ~]# systemctl start mysql
# 停止
[root@MYSQL-MASTER ~]# systemctl stop mysql
# 重启
[root@MYSQL-MASTER ~]# systemctl restart mysql
```



### 12.设置远程连接

```shell
[root@MYSQL-MASTER ~]# mysql -uroot -p'test1234'
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| yunwei_auto        |
+--------------------+
5 rows in set (0.00 sec)

mysql> use mysql;
Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
31 rows in set (0.00 sec)

mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
| localhost | yunwei_auto   |
+-----------+---------------+
4 rows in set (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON yunwei_auto.* TO 'yunwei_auto'@'%'IDENTIFIED BY 'test1234' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```



## 二、主从复制配置

### 1.配置hosts

```shell
############### 示例 ###############
192.168.19.164 MYSQL-MASTER
192.168.19.165 MYSQL-SLAVE
###################################
[root@MYSQL-MASTER ~]# hostnamectl set-hostname 'MYSQL-MASTER'
[root@MYSQL-SLAVE ~]# hostnamectl set-hostname 'MYSQL-SLAVE'

# 分别配置集群hosts文件
[root@MYSQL-MASTER ~]# vi /etc/hosts
192.168.19.164 MYSQL-MASTER
192.168.19.165 MYSQL-SLAVE
[root@MYSQL-SLAVE ~]# vi /etc/hosts
192.168.19.164 MYSQL-MASTER
192.168.19.165 MYSQL-SLAVE
```



### 2.主节点配置

#### 2.1 配置文件调整

```shell
[root@MYSQL-MASTER ~]# vi /etc/my.cnf
[mysqld]
# MySQL端口设置
port=3306

# 配置主从复制参数
# 设置主节点id
server-id=1
log-bin=mysql-bin
# 保留7天的二进制日志，防止磁盘被日志占满
expire-logs-days=7
# 不需要备份的数据库
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performation_schema
binlog-ignore-db=sys
# 需要做复制的数据库
binlog-do-db=yunwei_auto
binlog-do-db=nacos

# 禁用dns反查
skip-name-resolve

# MySQL安装路径
basedir=/usr/local/mysql-5.7.44

# MySQL数据存放路径
datadir=/data/mysql/data

# MySQL字节套sock存放路径
socket=/tmp/mysql.sock
socket=/data/mysql/data/mysql.sock

# MySQL日志存放
log-error=/data/mysql/mysqld.log

# MySQL服务PID存放路径
pid-file=/data/mysql/mysqld.pid

# MySQL字符集设置
character_set_server=utf8mb4

# MySQL最大连接数设置
max_connections=64

# innodb设置
innodb_buffer_pool_size = 256M
innodb_log_file_size = 128M
innodb_flush_method = O_DIRECT
innodb_flush_log_at_trx_commit = 0

# MySQL客户端设置
[client]
default-character-set=utf8mb4
socket=/data/mysql/data/mysql.sock
```



#### 2.2 重启服务

```shell
[root@MYSQL-MASTER ~]# systemctl restart mysql
[root@MYSQL-MASTER ~]# systemctl status mysql
● mysql.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/rc.d/init.d/mysql; bad; vendor preset: disabled)
   Active: active (running) since Fri 2023-11-10 14:21:14 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 13144 ExecStop=/etc/rc.d/init.d/mysql stop (code=exited, status=0/SUCCESS)
  Process: 13178 ExecStart=/etc/rc.d/init.d/mysql start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/mysql.service
           ├─13195 /bin/sh /usr/local/mysql-5.7.44/bin/mysqld_safe --datadir=/data/mysql/data --pid-file=/data/mysql/mysqld.pid
           └─13554 /usr/local/mysql-5.7.44/bin/mysqld --basedir=/usr/local/mysql-5.7.44 --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql-5.7.44/lib/plugin --user=mysql --log-error=/data/mysql/mysqld.log --pid-file=/data/my...

Nov 10 14:21:12 MYSQL-MASTER systemd[1]: Stopped LSB: start and stop MySQL.
Nov 10 14:21:12 MYSQL-MASTER systemd[1]: Starting LSB: start and stop MySQL...
Nov 10 14:21:14 MYSQL-MASTER mysql[13178]: Starting MySQL.. SUCCESS!
Nov 10 14:21:14 MYSQL-MASTER systemd[1]: Started LSB: start and stop MySQL.
```



#### 2.3 检查MySQL内部配置

```shell
[root@MYSQL-MASTER ~]# mysql -uroot -p'test1234'
# 设置读写有效操作
flush tables with read lock

# 创建从节点连接用户和授权
mysql> create user slaveuser@'%' identified by 'test1234';
mysql> grant replication slave on *.* to slaveuser@'%';
mysql> grant REPLICATION CLIENT on *.* to slaveuser@'%';
mysql> show grants for slaveuser@'%';
+-----------------------------------------------------------------------+
| Grants for slaveuser@%                                                |
+-----------------------------------------------------------------------+
| GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slaveuser'@'%' |
+-----------------------------------------------------------------------+
1 row in set (0.00 sec)
# 查看配置信息
mysql> select * from mysql.user\G;
*************************** 8. row ***************************
                  Host: %
                  User: slaveuser
           Select_priv: N
           Insert_priv: N
           Update_priv: N
           Delete_priv: N
           Create_priv: N
             Drop_priv: N
           Reload_priv: N
         Shutdown_priv: N
          Process_priv: N
             File_priv: N
            Grant_priv: N
       References_priv: N
            Index_priv: N
            Alter_priv: N
          Show_db_priv: N
            Super_priv: N
 Create_tmp_table_priv: N
      Lock_tables_priv: N
          Execute_priv: N
       Repl_slave_priv: Y
      Repl_client_priv: Y
      Create_view_priv: N
        Show_view_priv: N
   Create_routine_priv: N
    Alter_routine_priv: N
      Create_user_priv: N
            Event_priv: N
          Trigger_priv: N
Create_tablespace_priv: N
              ssl_type: 
            ssl_cipher: 
           x509_issuer: 
          x509_subject: 
         max_questions: 0
           max_updates: 0
       max_connections: 0
  max_user_connections: 0
                plugin: mysql_native_password
 authentication_string: *3D3B92F242033365AE5BC6A8E6FC3E1679F4140A
      password_expired: N
 password_last_changed: 2023-11-10 13:29:17
     password_lifetime: NULL
        account_locked: N

mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set (0.01 sec)

mysql> show master status;
+------------------+----------+-------------------+--------------------------------------------------+-------------------+
| File             | Position | Binlog_Do_DB      | Binlog_Ignore_DB                                 | Executed_Gtid_Set |
+------------------+----------+-------------------+--------------------------------------------------+-------------------+
| mysql-bin.000001 |      154 | yunwei_auto,nacos | mysql,information_schema,performation_schema,sys |                   |
+------------------+----------+-------------------+--------------------------------------------------+-------------------+
1 row in set (0.00 sec)

```



### 3.从节点配置

#### 3.1 配置文件调整

```shell
[root@MYSQL-SLAVE ~]# vi /etc/my.cnf
[mysqld]
# MySQL端口设置
port=3306

# 从节点配置
server-id=2

# 设置只读
#read_only=1

# 主节点IP地址
report-host=192.168.19.163

# 中继日志后缀名，默认在datadir目录下
relay-log=slave-relay-bin

# 开启从节点二进制日志记录
log-slave-updates=on

# 从节点禁止写，当从节点发生崩溃宕机时，会将未做完的中继日志删除并重新向主库获取日志，再次产生中继日志恢复，建议开启，若对MySQL优化则关闭
#relay_log_recovery=1

# 禁用dns反查
skip-name-resolve

# MySQL安装路径
basedir=/usr/local/mysql-5.7.44

# MySQL数据存放路径
datadir=/data/mysql/data

# MySQL字节套sock存放路径
socket=/tmp/mysql.sock
socket=/data/mysql/data/mysql.sock

# MySQL日志存放
log-error=/data/mysql/mysqld.log

# MySQL服务PID存放路径
pid-file=/data/mysql/mysqld.pid

# MySQL字符集设置
character_set_server=utf8mb4

# MySQL最大连接数设置
max_connections=64

# innodb设置
innodb_buffer_pool_size = 256M
innodb_log_file_size = 128M
innodb_flush_method = O_DIRECT
innodb_flush_log_at_trx_commit = 0

# MySQL客户端设置
[client]
default-character-set=utf8mb4
socket=/data/mysql/data/mysql.sock
```



#### 3.2 重启服务

```shell
[root@MYSQL-SLAVE ~]# systemctl restart mysql
[root@MYSQL-SLAVE ~]# systemctl status mysql
● mysql.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/rc.d/init.d/mysql; bad; vendor preset: disabled)
   Active: active (running) since Fri 2023-11-10 15:01:09 CST; 5s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 10902 ExecStop=/etc/rc.d/init.d/mysql stop (code=exited, status=0/SUCCESS)
  Process: 10932 ExecStart=/etc/rc.d/init.d/mysql start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/mysql.service
           ├─10949 /bin/sh /usr/local/mysql-5.7.44/bin/mysqld_safe --datadir=/data/mysql/data --pid-file=/data/mysql/mysqld.pid
           └─11248 /usr/local/mysql-5.7.44/bin/mysqld --basedir=/usr/local/mysql-5.7.44 --datadir=/data/mysql/data --plugin-dir=/usr/local/mysql-5.7.44/lib/plugin --user=mysql --log-error=/data/mysql/mysqld.log --pid-file=/data/my...

Nov 10 15:01:08 MYSQL-SLAVE systemd[1]: Stopped LSB: start and stop MySQL.
Nov 10 15:01:08 MYSQL-SLAVE systemd[1]: Starting LSB: start and stop MySQL...
Nov 10 15:01:09 MYSQL-SLAVE mysql[10932]: Starting MySQL. SUCCESS!
Nov 10 15:01:09 MYSQL-SLAVE systemd[1]: Started LSB: start and stop MySQL.
```



#### 3.3 检查MySQL内部配置

```shell
# 从节点配置
# 先查看master的 MASTER_LOG_FILE 和 MASTER_LOG_POS 对应的值
[root@MYSQL-SLAVE ~]# mysql -uroot -p'test1234'

mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.19.163',MASTER_USER='slaveuser',MASTER_PASSWORD='test1234',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=154;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)


mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.19.163
                  Master_User: slaveuser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 527
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 0974e168-7ec4-11ee-9d22-000c2943dc11
             Master_Info_File: /data/mysql/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

