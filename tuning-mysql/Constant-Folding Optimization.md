## Constant-Folding 优化

Constant-Folding 是 MySQL-8.0.x 的新功能，通过这个可以优化掉 MySQL 在 Server 层进行比较的这一环节；从执行计划来看就是少了 `Using where` 。 

要讲清楚这个还要从 MySQL-5.7 说起。

![sql](static/2020-27/sqlpy-const-floding.jpg)

---

## MySQL-5.7 的行为
1、建表 & 造数据。
```sql
create table t(x tinyint not null);

-- 4194304 行数据
insert into t(x) values(1),(2),(3);
```
2、where 条件的范围比列所能表示的范围要大时，MySQL-5.7 的处理行为如下。
```sql
mysql> select * from t where x < 256;                                                             
+---+
| x |
+---+
| 1 |
| 2 |
| 3 |
+---+
3 rows in set (0.00 sec)

mysql> explain select * from t where x < 256 \G                                                   
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 3
     filtered: 33.33
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```
因为 tinyint 的范围在 `-128 ~ 127` 之间，而 where 条件是 `x < 256 `，所以表中的所有行都被命中了。虽然 MySQL-5.7 正确的返回了表中所有的行，但是在执行计划中看到使用了 `Using where`，说明它有在 Server 做 `x < 256` 这个比较，当前场景下这个比较是没有必要的。

Constant-Folding 优化就是要把这种没有必要的步骤去掉。

google-adsense

---

## MySQL-8.0 的行为
1、建表 & 造数据。
```sql
create table t(x tinyint not null);

-- 4194304 行数据
insert into t(x) values(1),(2),(3);
```
2、where 条件的范围比列所能表示的范围要大时，MySQL-8.0 的处理行为如下。
```sql
mysql> select * from t where x < 256;                                                             
+---+
| x |
+---+
| 1 |
| 2 |
| 3 |
+---+
3 rows in set (0.00 sec)

mysql> explain select * from t where x < 256 \G                                                   
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

```
可以看到 MySQ-8.0 优化掉了没有必要的 Using where 。


---

## 性能比较
5.7 的耗时如下。
```sql
mysql> explain select * from tempdb.t where x < 256 \G                                            
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4188092
     filtered: 33.33
        Extra: Using where
1 row in set, 1 warning (0.00 sec)

mysql> select * from tempdb.t where x < 256 ;                                                     
4194304 rows in set (2.33 sec)

```

---

8.0 的耗时如下。
```sql
mysql> explain select * from tempdb.t where x < 256 \G                                            
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4188092
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

mysql> select * from tempdb.t where x < 256 ;
4194304 rows in set (1.84 sec)
```

---