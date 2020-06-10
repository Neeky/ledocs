## is null 优化
![sqlpy-is-null](static/2020-22/sqlpy-0610-isnull.jpg)

简单来讲就是，如果一个列在定义时被指定为 `not null`，但是在查询的 where 条件中是 `is null`，那么 MySQL 就可以直接判定这个一定为 False，并返回空结果。

下面我们会先看一下可以使用到 is null 优化的场景，再来看一下对于那些用不到 is null 优化的场景怎么能让它用上这个优化。

---

## 表结构和数据
实验环境的表结构和数据如下。
```sql
mysql> show create table t;                                                                      
+------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                               |
+------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `i0` int DEFAULT '0',
  `i1` int NOT NULL,
  `i2` int NOT NULL,
  `i3` int NOT NULL,
  `c0` varchar(128) NOT NULL,
  `c1` varchar(128) NOT NULL,
  `f0` float NOT NULL,
  `f1` float NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_i012` (`i0`,`i1`,`i2`)
) ENGINE=InnoDBB AUTO_INCREMENT=1120001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select count(*) from t;                                                                   
+----------+
| count(*) |
+----------+
|  1120000 |
+----------+
1 row in set (1.12 sec)
```

google-adsense

---


## 主键的情况
第一范式实体完整性，也就是说主键不可能为 null，就算我们在 create table 的时候没有指定，MySQL 也会为我们自动加上。也就是说对于主键来说天然的就可以用上 is null 优化。
```sql
mysql> explain select * from t where id is null;                                                 
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
1 row in set, 1 warning (0.00 sec)
```
可以看到 MySQL 识别出来了这是一个不可能成立的条件，直接给它打上了 `Impossible WHERE`。

---

## 其它情况
像 t 表中的 i0 列，它在定义时指定的默认为 0 ，如果我们又确认程序不可能给它插入一行 null 值，那么只要表结构改造一下就可以用的上 is null 优化，还是先来看一下用不到的情况。

```sql
mysql> select * from t where i0 is null;
Empty set (0.00 sec)

mysql> explain select * from t where i0 is null;                                                 
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-----------------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | t     | NULL       | ref  | idx_i012      | idx_i012 | 5       | const |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
从执行计划来看并没有出现 `Impossible WHERE`，那么 MySQL 在真正执行的时候应该还是去 idx_i012 索引上找了一下。 想要用上 is null 优化，只要把列声明为 not null 就行。

```sql
mysql> alter table t modify i0 int not null default 0;

mysql> explain select * from t where i0 is null;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
1 row in set, 1 warning (0.00 sec)
```
可以看到执行计划已经显示 `Impossible WHERE` 了。

---


## 总结

像 is null 这样的优化也是在读和写之间做性能的平衡，加入更多的约束条件那么写的时候就会做更多的检查，好处就是在读的时候对于一些明显不可能成立的条件就可以直接返回结果了。

经验上来看 MySQL 的单条写操作通常用时在 2ms 左右就可以完成，而大多数的场景又是写少读多，所以建议把必要的约束都定义上。

---