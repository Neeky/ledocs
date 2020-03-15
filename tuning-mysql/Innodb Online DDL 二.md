## Innodb-Online-DDL与DML的冲突点

DML 能执行的一个必要条件是它要能拿到表的 `metadata` 读锁，而 DDL 是要改表定义所以它要拿到表的 `metadata` 写锁。可以看出 DDL 和 DML 是有可能起冲突的，lock 选项可以指定 DDL 的上锁模式，尽可能的让 DDL 执行的过程中不阻塞 DML。

![innodb-online-ddl](static/2020-11/innodb-online-ddl-001.png)

---

## metadata-lock的三个上锁阶段
1、准备阶段: 这个阶段对元数据上共享锁

2、执行阶段: 如果 clock 选项没有指定是 exclusive 那么这个阶段不会把元数据共享锁升级到排他锁

3、提交新的表定义阶段: 这个阶段一定会把元数据共享升级为元数据排他锁

---

## exclusive模式
session A 以排他的方式执行 DDL 操作，用来阻塞其它的 DML 操作
```sql
alter table t add column c_001 int ,algorithm=copy,lock=exclusive;
```
session B 执行 DML 操作
```sql
select c_001 from t limit 1;
```
session C 观察这个时候数据库的表现
```sql
show processlist;
+----+------+-----------------+--------+---------+------+---------------------------------+-------------------------------------------------------------------+
| Id | User | Host            | db     | Command | Time | State                           | Info                                                              |
+----+------+-----------------+--------+---------+------+---------------------------------+-------------------------------------------------------------------+
|  8 | root | 127.0.0.1:41914 | tempdb | Query   |   14 | copy to tmp table               | alter table t add column c_001 int ,algorithm=copy,lock=exclusive |
| 11 | root | 127.0.0.1:41920 | tempdb | Query   |    5 | Waiting for table metadata lock | select c_001 from t limit 1                                       |
| 12 | root | 127.0.0.1:41922 | tempdb | Query   |    0 | starting                        | show processlist                                                  |
+----+------+-----------------+--------+---------+------+---------------------------------+-------------------------------------------------------------------+
3 rows in set (0.04 sec)
```
---

## shared模式
shared模式只在 DDL　的最后阶段(改表定义)才会升级到排他锁，所以整体上看感觉不到 meatadata 引起的阻塞
```sql
alter table t drop column c_001 int ,algorithm=copy,lock=exclusive;
```
session B 执行 DML 操作
```sql
select c_001 from t limit 1;
```
session C 观察数据库的状态
```sql

show processlist;
+----+------+-----------------+--------+---------+------+-------------------+------------------------------------------------------------+
| Id | User | Host            | db     | Command | Time | State             | Info                                                       |
+----+------+-----------------+--------+---------+------+-------------------+------------------------------------------------------------+
|  8 | root | 127.0.0.1:41914 | tempdb | Query   |    6 | copy to tmp table | alter table t drop column c_001,algorithm=copy,lock=shared |
| 11 | root | 127.0.0.1:41920 | tempdb | Sleep   |    3 |                   | NULL                                                       |
| 12 | root | 127.0.0.1:41922 | tempdb | Query   |    0 | starting          | show processlist                                           |
+----+------+-----------------+--------+---------+------+-------------------+------------------------------------------------------------+
3 rows in set (0.01 sec)
        
```
---

## default模式
这个模式就是 mysql 的默认模式，这一模式就是让 mysql 自动决定，但它会倾向于尽可能的不阻塞 DML 语句的执行。

---

