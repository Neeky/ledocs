## 背景
监控系统告警 MySQL 数据库复制中断，error-id = 1594 ，登录上主机看到如下报错信息
```sql
Last_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
Skip_Counter: 0
```
![Relay-log-read-failure](static/2020-12/Relay-log-read-failure.png)


---

## 原因
slave 的 IO 线程读取 master 的 binlog 保存到本地的 relaylog 中。如果 slave 在写 relaylog 的过程中 linux 宕机了，这就有可能导致 relaylog 中记录的 event 不完整，当主机重启、mysql数据库重启、SQL 线程开始读取 relaylog 的时候就会报这个错。
google-adsense

---

## 解决方案
知道问题是由于 slave 本机的 relay-log 不完整导致的错误，所以要修复问题就变成了让 slave 删除掉本机的 relay-log 然后去 master 那里同步一份新的就行了。要完成这个操作不正是 `chage master to` 要做的事吗？
```sql
stop slave;
-- change master 会清理当前的 relaylog , 由于我开了 gtid 就不用指定文件名和位置了
change master to 
    master_auto_position=1;

start slave;
```
---


## 检查
修复完成之后 MySQL 数据库的状态如下
```sql
MySQL [(none)]> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.80.100
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.002222
          Read_Master_Log_Pos: 65637801
               Relay_Log_File: relay-bin.000224
                Relay_Log_Pos: 59664068
        Relay_Master_Log_File: mysql-bin.002222
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

---

