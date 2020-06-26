## insert ignore 要解决的问题
当给定的行在目标表中已经存在时就跳过对这一行的插入。

![sqlpy](static/2020-26/sqlpy-insert-ignore.jpg)

---


## 效果演示
1、建表。
```sql
mysql> create table t( id int not null primary key auto_increment,v int);
Query OK, 0 rows affected (0.01 sec)
```

google-adsense

---

2、插入重复数据。
```sql
mysql> insert ignore into t(id,v) values(1,100);
Query OK, 1 row affected (0.01 sec)

mysql> insert ignore into t(id,v) values(1,100);
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show warnings;                                                                            
+---------+------+-----------------------------------------+
| Level   | Code | Message                                 |
+---------+------+-----------------------------------------+
| Warning | 1062 | Duplicate entry '1' for key 't.PRIMARY' |
+---------+------+-----------------------------------------+
1 row in set (0.01 sec)

mysql> select * from t;                                                                          
+----+------+
| id | v    |
+----+------+
|  1 |  100 |
+----+------+
1 row in set (0.00 sec)
```
可以看到当数据重复时 insert ignore 不会报错，而是报一个 warning ，并且重复的数据并不会被插入。

---

3、对于一次插入多行的情况 insert ignore 也能处理。
```sql
mysql> insert ignore into t(id,v) values(1,100),(2,200);                                         
Query OK, 1 row affected, 1 warning (0.00 sec)
Records: 2  Duplicates: 1  Warnings: 1

mysql> select * from t;
+----+------+
| id | v    |
+----+------+
|  1 |  100 |
|  2 |  200 |
+----+------+
2 rows in set (0.00 sec)
```

---

## 注意事项
insert ignore 是通过“主键”，“唯一键” 来判断行是否重复的，如果在一个表上这两类键都没有，那么 insert ignore 就不起作用。
```sql
mysql> drop table t;
Query OK, 0 rows affected (0.01 sec)

mysql> create table t(v int);                                                                    
Query OK, 0 rows affected (0.01 sec)

mysql> insert into t(v) values(100);                                                             
Query OK, 1 row affected (0.01 sec)

mysql> insert into t(v) values(100);
Query OK, 1 row affected (0.00 sec)

mysql> insert into t(v) values(100);
Query OK, 1 row affected (0.01 sec)

mysql> select * from t;                                                                          
+------+
| v    |
+------+
|  100 |
|  100 |
|  100 |
+------+
3 rows in set (0.00 sec)
```


---