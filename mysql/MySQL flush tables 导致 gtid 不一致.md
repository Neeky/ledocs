## 背景
收到监控发来的告警说是 slave 上多出了几个 gtid ，第一个念头是不好有人在 slave 上进行了写入操作，数据已经不一致了。

![sqlpy-gtid](static/2020-32/sqlpy-gtid.jpg)


---

## 检查
按我们的标准 slave 上的 read only 都是打开的才对，不应该有人可以写入;检查结果证明也确实如此。
```sql
mysql> show global variables like '%read%only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    | -- read_only ON
| super_read_only       | ON    | -- super_read_only ON
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.01 sec)
```

---

## 处理
根据多出的 gtid 从 binlog 中解析出 SQL 发现是 `flush tables`；以下是在测试环境上做的复现。
```sql
mysql> select @@version;                                                                         
+-----------+
| @@version |
+-----------+
| 8.0.20    |
+-----------+
1 row in set (0.00 sec)

mysql> show master status \G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 593
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 1419c78b-c372-11ea-b286-000c29cb87a3:1-3
1 row in set (0.00 sec)

mysql> flush tables;
Query OK, 0 rows affected (0.00 sec)

mysql> show master status \G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 740
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 1419c78b-c372-11ea-b286-000c29cb87a3:1-4
1 row in set (0.00 sec)
```

`flush tables` 确实导致 gtid 上涨了。

---

## 规避方案
可以用 `flush local tables` 来代替 `flush tables` 这样 gtid 就不会涨了，就算是在 slave 上也不会引起问题。
```sql
mysql> show master status \G                                                                     
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 740
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 1419c78b-c372-11ea-b286-000c29cb87a3:1-4
1 row in set (0.00 sec)

mysql> flush local tables; -- 这样 gtid 不会上涨
Query OK, 0 rows affected (0.00 sec)

mysql> show master status \G
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 740
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 1419c78b-c372-11ea-b286-000c29cb87a3:1-4
1 row in set (0.00 sec)
```

---


