## 函数调用优化
当 where 条件中出现非确定函数时使用 MySQL 用不了索引，就像下面的例子。
```sql
mysql> show create table t;                                                                      
+--------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                      |
+--------------------------------------------------------------------------------------------------------+
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
+--------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> explain select * from t where id = rand() + 1;
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows  | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 19966 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+-------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
可以看到执行计划中的 type 为 ALL ，说明 MySQL 要做全表扫描才能找到目录数据。

![sqlpy](static/2020-22/sqlpy-0610-a.jpg)

---

## 问题分析与解决
一个函数是确定函数还是非确定函数是根据它的返回值来说的，如果传入函数的参数相同返回的结果也相同，那么这样的函数叫确定函数，反之叫不确定函数，不确定函数中最有名的应该数 `rand()`随机函数和 `uuid()` 函数了。 

回到问题本身，既然是由于非确定函数造成的，那在条件允许的情况下把它变成确定的不就行了吗？

google-adsense

---

## 例子
对于上面的例子来说我们可以先把值求出来，再传给 where 子句，这样就由非确定函数转换为常量了，也就把值确定下来了。
```sql
mysql> set @tid =  rand() + 1;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where id = @tid;                                                  
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                               |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE noticed after reading const tables |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------------------------------+
1 row in set, 1 warning (0.00 sec)
```
可以看到优化器立马就知道了不可能有结果返回；原因是 id 是整数，而 @tid 是一个浮点数，所以不可能相等，也就是表中不可能会有这样的 id 。


---


## 参考
往期文章 [MySQL 数据库 where 条件可以用函数吗](/blogs/162805673)

官方文档[Function Call Optimization](https://dev.mysql.com/doc/refman/8.0/en/function-optimization.html)

---