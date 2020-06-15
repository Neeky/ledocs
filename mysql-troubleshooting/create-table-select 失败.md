## 背景
有同事想复制表`t`的结构和数据到一个新的表上去，但是他执行 SQL 失败。
```sql
mysql> create table t_s as select * from t;                                                      
ERROR 1786 (HY000): Statement violates GTID consistency: CREATE TABLE ... SELECT.
```

![sqlpy](static/2020-25/sqlpy-create-table.jpg)

---

## 原因分析
形如`create table xxx as select xxx;`这样的 SQL 在 MySQL 内部会被处理成两个部分。

第一部分是 `create table xxx`。
```sql
CREATE TABLE `t_s` (
  `id` int NOT NULL DEFAULT '0',
  `x` int DEFAULT NULL
)
```
第二部分是 `insert into xxx`。
```sql
INSERT INTO `tempdb`.`t_s`
SET
  @1=1 /* INT meta=0 nullable=0 is_null=0 */
  @2=100 /* INT meta=0 nullable=1 is_null=0 */
```
由于第一部分是 DDL 语句，所以它会被隐式的提交，这个就导致了事实上形成了两个事务，如果实例有打开 gtid 那么这两个事务就会使用相同的 gtid。

当 binlog 同步到 slave 后，第一个事务可以正常的完成，第二个事务就不会被执行了，因为这个事务对应的 gtid 已经存在了；这个就会导致主从数据的不一致。

这就是 MySQL 在开启 gtid 时禁用掉这个语句的原因。

google-adsense

---

## 复现
在一个关闭了 gtid 的实例上执行下面的代码，完成之后观察 binlog 就会发现确实是被分成了两个事务。

```sql
mysql> show global variables like 'gtid_mode';                                                  
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> create table t_s as select * from t; 
Query OK, 1 row affected (1.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t;                                                                          
+----+------+
| id | x    |
+----+------+
|  1 |  100 |
+----+------+
1 row in set (0.00 sec)

mysql> select * from t_s;                                                                        
+----+------+
| id | x    |
+----+------+
|  1 |  100 |
+----+------+
1 row in set (0.00 sec)


```
binlog 的内容如下。
```sql
SET TIMESTAMP=1588865633/*!*/;
/*!80013 SET @@session.sql_require_primary_key=0*//*!*/;
CREATE TABLE `t_s` (
  `id` int NOT NULL DEFAULT '0',
  `x` int DEFAULT NULL
)
/*!*/;
# at 574
#200507 23:33:53 server id 2099  end_log_pos 649        Anonymous_GTID  last_committed=2        sequence_number=3        rbr_only=yes    original_committed_timestamp=1588865633066696   immediate_commit_timestamp=1588865633066696      transaction_length=284
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
# original_commit_timestamp=1588865633066696 (2020-05-07 23:33:53.066696 CST)
# immediate_commit_timestamp=1588865633066696 (2020-05-07 23:33:53.066696 CST)
/*!80001 SET @@session.original_commit_timestamp=1588865633066696*//*!*/;
/*!80014 SET @@session.original_server_version=80020*//*!*/;
/*!80014 SET @@session.immediate_server_version=80020*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 649
#200507 23:33:53 server id 2099  end_log_pos 858        Transaction_Payload             payload_size=180 compression_type=ZSTD   uncompressed_size=243
# Start of compressed events!
# at 858
#200507 23:33:53 server id 2099  end_log_pos 858        Query   thread_id=77    exec_time=0     error_code=0
SET TIMESTAMP=1588865633/*!*/;
BEGIN
/*!*/;
# at 858
#200507 23:33:53 server id 2099  end_log_pos 858        Rows_query
# create table t_s as select * from t
# at 858
#200507 23:33:53 server id 2099  end_log_pos 858        Table_map: `tempdb`.`t_s` mapped to number 157
# at 858
#200507 23:33:53 server id 2099  end_log_pos 858        Write_rows: table id 157 flags: STMT_END_F

BINLOG '
YSq0Xh0zCAAANwAAAAAAAACAACNjcmVhdGUgdGFibGUgdF9zIGFzIHNlbGVjdCAqIGZyb20gdA==
YSq0XhMzCAAAMAAAAAAAAAAAAJ0AAAAAAAEABnRlbXBkYgADdF9zAAIDAwACAQEA
YSq0Xh4zCAAAKAAAAAAAAAAAAJ0AAAAAAAEAAgAC/wABAAAAZAAAAA==
'/*!*/;
### INSERT INTO `tempdb`.`t_s`
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=100 /* INT meta=0 nullable=1 is_null=0 */
# at 858
#200507 23:33:53 server id 2099  end_log_pos 858        Xid = 4682
COMMIT/*!*/;
# End of compressed events!
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
```
---

## 解决方案
知道原因后解决起来就方便了，我们只要把 SQL 拆成两个不就有两个 gtid 了。
```sql
mysql> show tables;
+------------------+
| Tables_in_tempdb |
+------------------+
| t                |
+------------------+
1 row in set (0.00 sec)

mysql> create table t_s like t;                                                                  
Query OK, 0 rows affected (0.02 sec)

mysql> insert into t_s(id,x) select id,x from t;                                                 
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t;                                                                          
+----+------+
| id | x    |
+----+------+
|  1 |  100 |
+----+------+
1 row in set (0.00 sec)

mysql> select * from t_s;                                                                        
+----+------+
| id | x    |
+----+------+
|  1 |  100 |
+----+------+
1 row in set (0.00 sec)

```


---