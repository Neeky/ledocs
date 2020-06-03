## 背景
近期有人向我咨询，`alter table xxx drop partition xxx` 要有什么权限才行，一时还真没答上来，不过我可以做实验把它搞出来。先说结论吧，要想成功执行这条语句我们需要 `alter`和 `drop` 权限。


---

## 实验步骤
用实验的方式一步步找出 `alter table drop partition` 要什么权限。

1、用超级用户完成建表和建用户。
```sql
use tempdb;
-- 建表
create table t (
  year_col int default null,
  v int default null
)
partition by range (year_col)
    (
    partition p1 values less than (1995),
    partition p2 values less than (1999),
    partition p3 values less than (2002),
    partition p4 values less than (2006),
    partition p5 values less than maxvalue
    );

-- 建用户
create user ddluser@'127.0.0.1' identified by '123456';
grant select on tempdb.t to ddluser@'127.0.0.1';
```
---

2、用 ddluser 登录进数据库并执行删除分区操作。
```sql
-- 确认用户与权限
mysql> show grants;                                                                              
+-------------------------------------------------------+
| Grants for ddluser@127.0.0.1                          |
+-------------------------------------------------------+
| GRANT USAGE ON *.* TO `ddluser`@`127.0.0.1`           |
| GRANT SELECT ON `tempdb`.`t` TO `ddluser`@`127.0.0.1` |
+-------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> alter table t drop partition p1;
ERROR 1142 (42000): DROP, ALTER command denied to user 'ddluser'@'127.0.0.1' for table 't'
```
可以看到在删除分区时报错了，并提示 alter, drop 权限不足。

---

3、按提示给用户加上 alter 和 drop 权限。
```sql
-- 用超级用户给 ddluser 用户加权限
mysql> grant alter,drop on tempdb.t to ddluser@'127.0.0.1';                                      
Query OK, 0 rows affected (0.00 sec)
```
发现一个彩蛋，MySQL-8.0.20 给用户增加新的权限不用重新连接就能生效。

---
4、执行删除分区的语句。
```sql
mysql> show grants;
+--------------------------------------------------------------------+
| Grants for ddluser@127.0.0.1                                       |
+--------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `ddluser`@`127.0.0.1`                        |
| GRANT SELECT, DROP, ALTER ON `tempdb`.`t` TO `ddluser`@`127.0.0.1` |
+--------------------------------------------------------------------+
2 rows in set (0.01 sec)

mysql> alter table t drop partition p1;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table t;                                                                      
+-------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                          |
+-------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `year_col` int DEFAULT NULL,
  `v` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
/*!50100 PARTITION BY RANGE (`year_col`)
(PARTITION p2 VALUES LESS THAN (1999) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2002) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (2006) ENGINE = InnoDB,
 PARTITION p5 VALUES LESS THAN MAXVALUE ENGINE = InnoDB) */ |
+-------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

---
