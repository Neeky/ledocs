## 背景
磁盘空间不足了，不处理吧一旦出了事，说出来又非常难听；之前的处理手法是把过期不再需要的数据删除掉，然后 `alter table xx engine=innodb;` 来释放空间。

最近突然想到我是不是把 MySQL 的数据压缩功能给忘记了，那 MySQL 的数据压缩比有多少呢？于是我在测试环境试了一下。

![sqlpy](static/2020-30/sqlpy-row-format.jpg)

---

## 环境介绍
1、数据库表结构如下。
```sql
mysql> show create table t;                                                                      
+-----------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+-----------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `i0` int NOT NULL,
  `i1` int NOT NULL,
  `i2` int NOT NULL,
  `i3` int NOT NULL,
  `i4` int NOT NULL,
  `i5` int NOT NULL,
  `i6` int NOT NULL,
  `i7` int NOT NULL,
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
) ENGINE=InnoDB AUTO_INCREMENT=80001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-----------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

空间占用情况如下。

```bash
ll t.ibd
-rw-r-----. 1 mysql3306 mysql 348M 7月  23 18:09 t.ibd
```

---

## 开启 MySQL 的表压缩功能
表压缩功能在 SQL 语句里的名字叫 `row_format` ，官方在文档中也特意的指出了这个名字不合适，但是没办法我们要用还是要这样用。
```sql
mysql> alter table t row_format=compressed;  
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
压缩后的空间大小。
```bash
ll 
-rw-r-----. 1 mysql3306 mysql 300M 7月  23 18:12 t.ibd
```

---


## 收益

在我的测试环境下大约可以节约 13.7% 的空间，由于数据样本的差异性，不保证你的数据也有这个压缩比。

|**压缩前大小**|**压缩后大小**|**空间占用减小**|
|------------|-------------|--------------|
|348M        | 300M        |13.7%(48/348) |
|||

---