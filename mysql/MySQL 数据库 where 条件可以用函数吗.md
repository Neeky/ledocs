## 问题
江湖上说 MySQL 数据库不要在 where 条件里使用函数不然就不能用索引了，这个是值的吗？真的不能在 where 子句里使用函数了吗？

---

## 建表造数据
创建 tempdb.t 表并向其插入 2w 行测试数据。
```bash
mtls-perf-bench --host=127.0.0.1 --port=3306 --user=root --password=dbma@0352 --ints=4 --floats=2 --varchars=2 create

mtls-perf-bench --host=127.0.0.1 --port=3306 --user=root --password=dbma@0352    --ints=4 --floats=2 --varchars=2 --parallel=4 --rows=20000 insert
```
检查是否成功。
```sql
mysql> show create table tempdb.t;                                                                
+------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                             |
+------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `i0` int NOT NULL,
  `i1` int NOT NULL,
  `i2` int NOT NULL,
  `i3` int NOT NULL,
  `c0` varchar(128) NOT NULL,
  `c1` varchar(128) NOT NULL,
  `f0` float NOT NULL,
  `f1` float NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=20001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select count(*) from tempdb.t;
+----------+
| count(*) |
+----------+
|    20000 |
+----------+
1 row in set (0.06 sec)
```
在 i3 列上加上索引。
```sql
mysql> alter table t add index idx_i3(i3);                                                        
Query OK, 0 rows affected (0.05 sec)
```

google-adsense

---

## 无法使用索引的情况
先确认一下索引是有效的。
```sql
mysql> explain select * from t where i3 = 1;                                                      
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i3        | idx_i3 | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```
从执行计划可以看出，符合条件的情况下是可以用索引的；再回到“where 条件中不能使用函数” 的话题，来看一个用不了索引的情况。
```sql
mysql> explain select * from t where i3 = abs(i3);                                                
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 19890 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
1 row in set, 1 warning (0.01 sec)
```
可以看到现在 where 条件里出现了函数，执行计划表现出了全表扫描。可以说是因为使用了函数所以就不走索引了吗？还不能，不信你看下面的例子。

---

## 可以使用索引的情况
为了证明“使用函数和不能使用索引”之间没有什么必然关系，我们看下面这个例子。
```sql
mysql> explain select * from t where i3 = year(now());                                            
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i3        | idx_i3 | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```
可以看到，这下使用了索引。

---

## 结论
where 条件中使用函数并不一定可以导出 sql 使用不了索引，关键还是要看函数的返回值是可以提交就知道，还要是等到读出行中的数据后才能知道。如果是前者那么 MySQL 可以使用用索引，后者的话主没有办法了。

不过还是不推荐使用函数，因为这个通常会增大 MySQL 的计算量，计算应该在应用层完成，MySQL 擅长的是数据存取。

---