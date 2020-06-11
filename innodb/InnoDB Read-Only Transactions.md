## InnoDB Read-Only Transactions 优化
innodb 表中的每一行数据都有一个隐藏字段 `trx_id` ,这个字段用来标记是哪个事务最后更新了这一行数据。事务 id 由 innodb 分配，但是一个只读事务不可能修改任何行。

也就是说为一个只读事务分配事务 id 是在做无用功。 开启只读事务就可以让 innodb 跳过这一步。

![sqlpy](static/2020-24/sqlpy-readonly-trx.jpg)

---

## 使用只读事务
开启只读事务的语法也非常简单
```sql
-- 开启只读事务
mysql> start transaction read only;
Query OK, 0 rows affected (0.00 sec)

-- 在只读事务中查询是没有问题的
mysql> select * from tempdb.scores;
+----+--------------+-------+--------+-------------+
| id | student_name | math  | physis | total_score |
+----+--------------+-------+--------+-------------+
|  1 | tom          | 120.0 |  100.0 |       220.0 |
+----+--------------+-------+--------+-------------+
1 row in set (0.00 sec)

-- 在只读事务内部不能执行更新语句
mysql> update tempdb.scores set math=100 where id =1;
ERROR 1792 (25006): Cannot execute statement in a READ ONLY transaction.

mysql> commit;
```

google-adsense

---

## 官方文档
更多的内容可以参考官方的 [Optimizing InnoDB Read-Only Transactions](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html) 章节。
