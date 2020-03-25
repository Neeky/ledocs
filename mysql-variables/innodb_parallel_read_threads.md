## innodb_parallel_read_threads
聚集索引扫描时 innodb 可以通过多个线程并发的读取页面来做到更快的把页面读出来，从而提升查询效率，并发线程的数量就由 innodb_parallel_read_threads 这个参数控制。

![innodb_parallel_read_threads](static/2020-13/innodb_parallel_read_threads.png)

google-adsense

---

## 准备测试环境
虚拟机的配置如下 cpu:4核、mem:4G、innodb_buffer_pool_size:1G、mysql:8.0.18；通过 mysqltools-python 工具包来创建表结构并填充数据。
```bash
mtls-perf-bench --host=127.0.0.1 --user=root --password=dbma@0352 --port=3306 --ints=8 --floats=8 --varchars=4 --database=tempdb create

mtls-perf-bench --host=127.0.0.1 --user=root --password=dbma@0352 --port=3306 --ints=8 --floats=8 --varchars=4 --database=tempdb --parallel=16 --rows=800000 insert
            
```
表结构如下。
```sql
mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.18    |
+-----------+
1 row in set (0.00 sec)

show create table tempdb.t;

CREATE TABLE `t` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `i0` int(11) NOT NULL,
  `i1` int(11) NOT NULL,
  `i2` int(11) NOT NULL,
  `i3` int(11) NOT NULL,
  `i4` int(11) NOT NULL,
  `i5` int(11) NOT NULL,
  `i6` int(11) NOT NULL,
  `i7` int(11) NOT NULL,
  `c0` varchar(128) NOT NULL,
  `c1` varchar(128) NOT NULL,
  `c2` varchar(128) NOT NULL,
  `c3` varchar(128) NOT NULL,
  `f0` float NOT NULL,
  `f1` float NOT NULL,
  `f2` float NOT NULL,
  `f3` float NOT NULL,
  `f4` float NOT NULL,
  `f5` float NOT NULL,
  `f6` float NOT NULL,
  `f7` float NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|   800000 |
+----------+
1 row in set (1.04 sec)
```

---

## 测试场景
因为只有聚集索引扫描才会受益于这个参数，所以我选择了 `select count(*) from t;` 用来测试不同并发下 MySQL 的耗时。

---

## 一个并发
一个并发的情况下 MySQL 耗时 1.04s。
```sql
mysql> show global variables like 'innodb_parallel_read_threads';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_parallel_read_threads | 1     |
+------------------------------+-------+
1 row in set (0.01 sec)

mysql> select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|   800000 |
+----------+
1 row in set (1.04 sec)
 
```

---


## 两个并发
两个并发的情况下 MySQL 耗时 0.59s。
```sql
mysql> show global variables like 'innodb_parallel_read_threads';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_parallel_read_threads | 2     |
+------------------------------+-------+
1 row in set (0.01 sec)

mysql> select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|   800000 |
+----------+
1 row in set (0.59 sec)
```

---

## 四个并发
四个并发的情况下 MySQL 耗时 0.55s。
```sql
mysql> show global variables like 'innodb_parallel_read_threads';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_parallel_read_threads | 4     |
+------------------------------+-------+
1 row in set (0.01 sec)

mysql> select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|   800000 |
+----------+
1 row in set (0.55 sec)
```

---

## 八个并发
八个并发的情况下 MySQL 耗时 0.29s。
```sql
mysql> show global variables like 'innodb_parallel_read_threads';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_parallel_read_threads | 8     |
+------------------------------+-------+
1 row in set (0.01 sec)

mysql> select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|   800000 |
+----------+
1 row in set (0.29 sec)
```

---

## 结论
innodb 并行读线程数(innodb_parallel_read_threads) 可以明显的提高聚集索引扫描场景下的性能。