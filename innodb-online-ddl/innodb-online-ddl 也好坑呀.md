## 背景
前些天在给表 t 加列的时候 MySQL 报错，说是违反了唯一键约束，我根据报错的提示去表中查询发现只有一行数据满足条件，说明事实上并没有违反约束

![ddl-error](static/2020-8/innodb-online-ddl.png)

```sql
mysql> ALTER TABLE `tempdb`.`t` ADD `c15` TEXT,ADD `c16` BLOB;

ERROR 1062 (23000): Duplicate entry '1055352-11023-BwwxbxbZh' for key 'uidx_c1_c2_c3'
```

第一次看到这个报错吓我一跳，难不成数据库中的数据真的不一致了，保险起见我检查了一下；检查的结果是数据没有不一致(满足条件的只有一条)
```sql
select c1,c2,c3 from tempdb.t;
+----------+----------------+------------------+
| c1       | c2             | c3               |
+----------+----------------+------------------+
| 69640784 | 11023          | BwwxbxbZh        |
+----------+----------------+------------------+
1 row in set (0.00 sec)
```
google-adsense

---


## 突破
与负责内核开发的同事沟通后得知这个是 online-ddl 的一个限制，当时还觉得不可思议；于是再次仔细的阅读了官方文档确认是有这样一回事 [innodb-online-ddl-limiations](https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl-limitations.html) 。

![ddl-error](static/2020-8/ddl-error.png)

根据官方提供的信息可以知道，这种情况只有在 DDL 期间要执行 DML 时才有可能发生；这样的话解决办法也就有了，一来我们可以把 DML 阻塞住以此来保证 DDL 的成功执行，二来我们可以直接用 pt-online-schema-change 这个工具来解决。

google-adsense

---

## 解决方案
在上述的两个方案中，我最终还是选择了阻塞 DML 的方式来解决这次问题，原因就只有一个简单呀。

```sql
mysql> ALTER TABLE `tempdb`.`t` ADD `c15` TEXT,ADD `c16` BLOB,LOCK = EXCLUSIVE;
Query OK, 0 rows affected (17 min 10.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
-- 在元数据上加排他锁
```



---

