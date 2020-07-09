## long_query_time

MySQL 中 long_query_time 参数用于控制慢查询的阀值，也就是说当一个 SQL 的执行时间超过这个值的时候会就被记录到慢查询文件中去。

另外这个值是 session 级的，也就是说一个连接的慢查询阀值，在建立连接之后就确定了。换名话说，当我们执行 `set @@global.long_query_time = xxx` 时对已经存在的连接没有影响。

![sqlpy](static/2020-27/sqlpy-long-query-time.jpg)

---

## 验证更新 long_query_time 对长连接无效

下面我将通过一个实验来说明更新 long_query_time 对长连接无效的这个事。

第一步、session 1 连接上数据库，并创造出一条慢查询。
```sql
mysql> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
|               8 |
+-----------------+
1 row in set (0.00 sec)

mysql> show global variables like 'long_query_time';                                             
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 7.000000 |
+-----------------+----------+
1 row in set (0.00 sec)

mysql> select sleep(8);
+----------+
| sleep(8) |
+----------+
|        0 |
+----------+
1 row in set (8.01 sec)

```
---

这个时候在慢查询日志中可以看到如下内容。
```log
# Time: 2020-07-09T21:28:22.230819+08:00
# User@Host: root[root] @  [127.0.0.1]  Id:     8
# Query_time: 8.001278  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1594301294;
select sleep(8);
```

---

第二步、通过 session 2 把慢查询的阀值减小到 5 秒。
```sql
mysql> select connection_id();                                                                   
+-----------------+
| connection_id() |
+-----------------+
|              18 |
+-----------------+
1 row in set (0.00 sec)

mysql> set @@global.long_query_time=5;                                                           
Query OK, 0 rows affected (0.00 sec)

```
---

第三步、在 session 上再造一条慢查询，可是慢查询文件中不会记录新的内容，因为就在于 session 1 的慢查询阀值还是之前的 7s 。
```sql
mysql> select sleep(6);                                                                          
+----------+
| sleep(6) |
+----------+
|        0 |
+----------+
1 row in set (6.00 sec)
```
---