## 背景
MySQL-8.0.20 给 `change master to` 加了一个新参数，`require_table_primary_key_check`，于是决定在测试环境上看一下这个参数的影响。

|**IP地址**|**角色**|**MySQL版本**|
|---------|--------|----|
|172.16.192.100| master |MySQL-8.0.20|
|172.16.192.88 | salve  |MySQL-8.0.20|

![sqlpy](static/2020-24/sqlpy-primary.jpg)


---

## 搭建主从环境
建立 172.16.192.100:3306 到  172.16.192.88:3306 的数据同步关系。
```sql
 change master to 
    master_host="172.16.192.100",
    master_port=3306,
    master_user="repluser",
    master_password="dbma@0352",
    master_ssl = 1,
    master_auto_position=1,
    require_table_primary_key_check=ON;
start slave;
```
在 master 上建库建表。
```sql
mysql> create database tempdb;                                                                   
Query OK, 1 row affected (0.00 sec)

mysql> use tempdb;                                                                               
Database changed
mysql> create table t(x int);      -- 故意不带主键用于触发 slave 的验证。                                                              
Query OK, 0 rows affected (0.02 sec)
```
可以看到从库开启 `require_table_primary_key_check` 并不会影响到主库的提交，哪它的影响又在哪里呢？当 slave 检查到表没有主键时会自动停止复制，并报错。

google-adsense

---

## 检查 slave 
当主库的表没有主键时 slave 可以看到如下错误。
```sql
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.16.192.100
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 3750
                   Last_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction '08b17c86-9bde-11ea-ae1f-000c29e0ca28:2' at master log mysql-bin.000002, end_log_pos 517. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
```
worker 线程的错误信息是这样的。
```sql
Worker 1 failed executing transaction '08b17c86-9bde-11ea-ae1f-000c29e0ca28:2' at master log mysql-bin.000002, end_log_pos 517; Error 'Unable to create or change a table without a primary key, when the system variable 'sql_require_primary_key' is set. Add a primary key to the table or unset this variable to avoid this message. Note that tables without a primary key can cause performance problems in row-based replication, so please consult your DBA before changing this setting.' on query. Default database: 'tempdb'. Query: 'create table t(x int)' 
```

---

## 结论
当 slave 开启 `require_table_primary_key_check` 后，如果遇到了要处理的表没有主键就会直接报错，停止复制。

---

## 为什么需要这个功能
当 binlog 的格式是 `row` 表又没有主键，这种情况下主库上一个微小的 update 或 delete 都有可能在从库上造成非常大的延时(我见过延时几天的)。要从根本上解决这个问题当然是给表加上主键，`require_table_primary_key_check` 提供了一个发现问题的新路径，虽然这条路径不是特别友好。

---













