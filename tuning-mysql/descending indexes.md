## descending indexes 要解决的问题
descending indexes (倒排索引) 是为了让 order by xxx desc 执行的更加快而开发的新特性。

![sqlpy](static/2020-25/sqlpy-desc-index.jpg)


---

## 实验环境
表结构和数据量如下。
```sql
mysql> show create table t;                                                                      
+------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                                                             |
+------------------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `i0` int NOT NULL DEFAULT '0',
  `i1` int NOT NULL,
  `i2` int NOT NULL,
  `i3` int NOT NULL,
  `c0` varchar(128) NOT NULL,
  `c1` varchar(128) NOT NULL,
  `f0` float NOT NULL,
  `f1` float NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_i012` (`i0`,`i1`,`i2`),
) ENGINE=InnoDB AUTO_INCREMENT=1120001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select count(*) from t;                                                                   
+----------+
| count(*) |
+----------+
|  1120000 |
+----------+
1 row in set (0.94 sec)
```
google-adsense

---

## 使用非倒排索引的情况
执行计划和实际执行耗时如下。
```sql
mysql> explain select i0,i3 from t where i0=1 order by i0,i1 desc;
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+---------------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows  | filtered | Extra               |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+---------------------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012      | idx_i012 | 4       | const | 21374 |   100.00 | Backward index scan |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+---------------------+
1 row in set, 1 warning (0.00 sec)

mysql> select i0,i3 from t where i0=1 order by i0,i1 desc;
11200 rows in set (2.29 sec)
```

---

## 使用倒排索引的情况
执行计划和实际执行耗时如下。
```sql
-- 添加倒排索引(i1 desc)
alter table t add index ixd_0_desc1(i0,i1 desc);

mysql> explain select i0,i3 from t where i0=1 order by i0,i1 desc;
+----+-------------+-------+------------+------+----------------------+-------------+---------+-------+-------+----------+-------+
| id | select_type | table | partitions | type | possible_keys        | key         | key_len | ref   | rows  | filtered | Extra |
+----+-------------+-------+------------+------+----------------------+-------------+---------+-------+-------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012,ixd_0_desc1 | ixd_0_desc1 | 4       | const | 20864 |   100.00 | NULL  |
+----+-------------+-------+------------+------+----------------------+-------------+---------+-------+-------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select i0,i3 from t where i0=1 order by i0,i1 desc;
11200 rows in set (0.82 sec)
```

---

## 结论
在当前的场景下使用倒排索引可以把耗时从 2.29s 降到 0.82s ，大致提升 3 倍性能。

---