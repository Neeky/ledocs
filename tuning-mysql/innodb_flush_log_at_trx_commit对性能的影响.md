## 背景
突然想知道 innodb_flush_log_at_trx_commit 参数的不同取值，对写入的性能的影响是怎样的。于是我做了如下测试
![innodb_flush_log_at_trx_commit](static/2020-12/innodb_flush_log_at_trx_commit.png)

---

## 等于0的场景
关键参数的值如下
```sql
mysql> show global variables like 'innodb_flush_log_at_trx_commit';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 0     |
+--------------------------------+-------+
1 row in set (0.01 sec)

mysql> show global variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```
对数据库库进行写入压力测试，耗时 7.50s  tps 10671
```bash
mtls-perf-bench --host=127.0.0.1 --port=3306 --user=root --password=xxxx --ints=4 --floats=2 --varchars=2 --parallel=8 --rows=80000 insert
2020-03-17 16:49:02,380  mtls-perf-bench  2699  MainThread  INFO  start time = 1584434942.38085
2020-03-17 16:49:02,381  mtls-perf-bench  2699  MainThread  INFO  ****
2020-03-17 16:49:02,381  mtls-perf-bench  2699  MainThread  INFO  ****
2020-03-17 16:49:02,408  mtls-perf-bench  2700  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,408  mtls-perf-bench  2701  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,409  mtls-perf-bench  2702  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,411  mtls-perf-bench  2703  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,411  mtls-perf-bench  2704  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,415  mtls-perf-bench  2705  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,415  mtls-perf-bench  2706  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:02,416  mtls-perf-bench  2707  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:49:09,877  mtls-perf-bench  2699  MainThread  INFO  ****
2020-03-17 16:49:09,877  mtls-perf-bench  2699  MainThread  INFO  ****
2020-03-17 16:49:09,877  mtls-perf-bench  2699  MainThread  INFO  stop time = 1584434949.8771398
2020-03-17 16:49:09,877  mtls-perf-bench  2699  MainThread  INFO  TPS:10671.95 duration 7.50(s)
```

---

## 等于1的场景
关键参数的值如下
```sql
mysql> show global variables like 'innodb_flush_log_at_trx_commit';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.01 sec)

mysql> 
mysql> show global variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```
对数据库库进行写入压力测试，耗时 23.16s  tps 3454.25
```bash
mtls-perf-bench --host=127.0.0.1 --port=3306 --user=root --password=xxxx --ints=4 --floats=2 --varchars=2 --parallel=8 --rows=80000 insert
2020-03-17 16:45:48,679  mtls-perf-bench  2667  MainThread  INFO  start time = 1584434748.6791773
2020-03-17 16:45:48,679  mtls-perf-bench  2667  MainThread  INFO  ****
2020-03-17 16:45:48,679  mtls-perf-bench  2667  MainThread  INFO  ****
2020-03-17 16:45:48,705  mtls-perf-bench  2668  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,705  mtls-perf-bench  2669  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,706  mtls-perf-bench  2670  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,707  mtls-perf-bench  2671  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,709  mtls-perf-bench  2673  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,711  mtls-perf-bench  2672  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,713  mtls-perf-bench  2674  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:45:48,715  mtls-perf-bench  2675  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:46:11,839  mtls-perf-bench  2667  MainThread  INFO  ****
2020-03-17 16:46:11,839  mtls-perf-bench  2667  MainThread  INFO  ****
2020-03-17 16:46:11,839  mtls-perf-bench  2667  MainThread  INFO  stop time = 1584434771.8390312
2020-03-17 16:46:11,839  mtls-perf-bench  2667  MainThread  INFO  TPS:3454.25 duration 23.16(s)
```
---

## 等于2的场景
关键参数的值如下
```sql
mysql> show global variables like 'innodb_flush_log_at_trx_commit';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 2     |
+--------------------------------+-------+
1 row in set (0.00 sec)

mysql> show global variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```
对数据库库进行写入压力测试，耗时 12.84s  tps 6228.97
```
mtls-perf-bench --host=127.0.0.1 --port=3306 --user=root --password=dbma@0352 --ints=4 --floats=2 --varchars=2 --parallel=8 --rows=80000 insert
2020-03-17 16:47:46,560  mtls-perf-bench  2685  MainThread  INFO  start time = 1584434866.560359
2020-03-17 16:47:46,560  mtls-perf-bench  2685  MainThread  INFO  ****
2020-03-17 16:47:46,560  mtls-perf-bench  2685  MainThread  INFO  ****
2020-03-17 16:47:46,605  mtls-perf-bench  2687  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,606  mtls-perf-bench  2688  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,607  mtls-perf-bench  2689  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,608  mtls-perf-bench  2686  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,608  mtls-perf-bench  2690  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,609  mtls-perf-bench  2691  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,610  mtls-perf-bench  2692  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:46,611  mtls-perf-bench  2693  MainThread  INFO  sql statement: insert into tempdb.t (i0,i1,i2,i3,c0,c1,f0,f1) values(%s,%s,%s,%s,%s,%s,%s,%s)
2020-03-17 16:47:59,403  mtls-perf-bench  2685  MainThread  INFO  ****
2020-03-17 16:47:59,403  mtls-perf-bench  2685  MainThread  INFO  ****
2020-03-17 16:47:59,403  mtls-perf-bench  2685  MainThread  INFO  stop time = 1584434879.403576
2020-03-17 16:47:59,403  mtls-perf-bench  2685  MainThread  INFO  TPS:6228.97 duration 12.84(s)
```

---

## 总结
从耗时上来看 1,2,0 一依次减小

![](static/2020-12/innodb_flush_log_at_trx_commit-time-es.png)

---
