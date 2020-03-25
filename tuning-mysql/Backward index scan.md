## 背景
默认的索引扫描方向是由 -∞ --> +∞  如果我们需要的数据本身就落在索引的尾部，那么从 +∞  --> -∞ 的扫描方向会更加有效率。
感觉还是有点抽象，下面看一下我这几天遇到的一个 case ，我们的表上有一个自增的 id 列，这样新数据的 id 值就会比旧数据的 id 值大，业务会根据一定条件查询出满足条件的，最新的 10 行数据，最新的 10 行数据在 SQL 查询中可以用 `order by id desc limit 10` 来表达

![backward-index-scan](static/2020-11/Backward-index-scan.png)
google-adsense

---

## 环境准备
这是一个线上 case 直接使用线上的表结构，会对我有些影响，为了说明这个问题我决定自己造一个
```sql
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
    PRIMARY KEY (`id`),
    KEY `idx_i0_i1` (`i0`,`i1`)
  ) ENGINE=InnoDB;

-- 填充上数据 
```
google-adsense

---


## 5.7.26下的性能表现
在 `5.7.26` 版本不进行人为优化的性能表现如下
```sql
-- 10 行数据用时 0.22 秒
mysql> select * from t where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
10 rows in set (0.22 sec)

-- 执行计划如下
explain select * from t where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | index | idx_i0_i1     | PRIMARY | 4       | NULL | 1945 |     0.17 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```
可以看到 `key` 值为 `primary` 说明 MySQL 走了聚焦索引扫描，针对这种情况应该先查询`idx_i0_i1`然后再回表会好一些；这里给 `5.7.26` 加上索引提示来优化性能
```sql
-- 加上索引提示 
select * from t force index(idx_i0_i1) where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
10 rows in set (0.01 sec)

-- 查看有索引提示后的执行计划
explain select * from t force index(idx_i0_i1) where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+---------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+---------------------------------------+
|  1 | SIMPLE      | t     | NULL       | range | idx_i0_i1     | idx_i0_i1 | 4       | NULL | 4530 |    33.33 | Using index condition; Using filesort |
+----+-------------+-------+------------+-------+---------------+-----------+---------+------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
```
加上索引提示后`5.7.26`的耗时由 `0.22s` 下降到 `0.01s` 提升了 `22`倍。

---

## Backward-index-scan
Backward index scan 是 MySQL-8.0.x 针对上面场景的一个专用优化项，它可以从索引的后面往前面读，性能上比加索引提示要好的多
```sql
select * from t where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
10 rows in set (0.00 sec)

-- 看一下 8.0.16 下的执行计划
explain select * from t where i0 < 10000000 and i1 < 10000000 order by id desc limit 10;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                            |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------+
|  1 | SIMPLE      | t     | NULL       | index | idx_i0_i1     | PRIMARY | 4       | NULL | 1894 |     0.18 | Using where; Backward index scan |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+----------------------------------+
1 row in set, 1 warning (0.00 sec)
```
---

## 总结
由此我们把一个查询由`0.22s`先是通过索引提示优化到了`0.01s`，然后又基于`8.0.16`把查询优化到了`0.00s`，是重要的是`8.0.16`不用改 SQL，这些切的收益都来自于版本的升级。

---



