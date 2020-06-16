## index condition pushdown 的优化逻辑
index condition pushdown 可以优化特定场景下的 SQL 。如果一条 SQL 有二级索引可以用，那么 MySQL 会先读二级索引，再回表查询查询表中的其它的列，最后根据条件来确定对应的行在不在结果集里面。

如果直接通过比较索引的值就能确认行在不在结果集里面，那不是会快很多吗？ Using index condition 就是做的这个，前面讲的比较宽泛，要知道 Using index condition 适合于什么场景还是要看例子。

![sqlpy](static/2020-25/sqlpy-using-index-condition.jpg)

---

## 实验环境
实验环境的表结构和数据量如下。
```sql
mysql> show create table t;
+----------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                       |
+----------------------------------------------------------------------------------------------+
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
  KEY `idx_i012` (`i0`,`i1`,`i2`)
) ENGINE=InnoDB AUTO_INCREMENT=1120001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+----------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> select count(*) from t;
+----------+
| count(*) |
+----------+
|  1120000 |
+----------+
1 row in set (1.96 sec)
```

google-adsense

---

## index condition pushdown 的适用场景
1、像这条 SQL 就适用于 index condition pushdown。
```sql
mysql> select i3 from t where i0=1 and i2=1; 
```
---

2、在没有 index condition pushdow 优化的情况下，执行计划是这样的.
```sql
mysql> explain select i3 from t where i0=1 and i2=1;                                             
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows  | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012      | idx_i012 | 4       | const | 21374 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
key_len = 4 说明 MySQL 先读了二级索引 idx_i012 ，然后回表找到行的 i2 列的值，只有当这个值等于 1 时，这一行才会被查询出来。

这个查询的流程可以这样优化，不用回表去找 i2 的值了，因为 i2 的值在索引 idx_i012 上就有，也就是说只要过滤索引就能知道行在不在结果集中。

google-adsense

---

## index condition pushdown 的优化效果
1、没有 index condition pushdown 优化的执行耗时。
```sql
mysql> explain select i3 from t where i0=1 and i2=1;
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows  | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012      | idx_i012 | 4       | const | 21374 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> select i3 from t where i0=1 and i2=1;
1135 rows in set (3.12 sec)
```
---

2、有 index condition pushdown 优化的执行耗时。
```sql
mysql> explain select i3 from t where i0=1 and i2=1;                                             
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows  | filtered | Extra                 |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-----------------------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012      | idx_i012 | 4       | const | 21374 |    10.00 | Using index condition |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+-------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

mysql> select i3 from t where i0=1 and i2=1;
1135 rows in set (0.21 sec)
```


在使用 index condition pushdown 的情况下耗时从 3.12s 下降到了 0.21s ，性能提升 15 倍左右。

---

## index condition pushdown 的执行计划特征
如果一条 SQL 有使用 index condition pushdown 优化那么它的执行计划中将包含 `Using index condition` 。

---

## 启用和关闭 index condition pushdown

是否启用 index condition pushdown 是由优化器的`index_condition_pushdown`参数控制的。
```sql
SET optimizer_switch = 'index_condition_pushdown=off';
```

---
