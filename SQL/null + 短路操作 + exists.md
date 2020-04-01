## 概要
在前文 [null 导致空结果](/blogs/119435262) 中我们提到了 SQL 语言的三值逻辑，这次我将更加深入的讨论一下 null 的坑。

![exists-null](static/2020-14/exists-null.png)

google-adsense

---

## null 与谓词短路
“所有涉及到 null 的操作都会返回 null” 这句话不太对，原因是 SQL 在进行条件测试时是会短路的，看下面的例子。

`false and null `会短路所以返回 false。
```sql
-- and 短路的情况
mysql> select true and null,false and null;
+---------------+----------------+
| true and null | false and null |
+---------------+----------------+
|          NULL |              0 |
+---------------+----------------+
1 row in set (0.00 sec)
```
`true or null` 也会短路所以会返回 true。
```sql
-- or 短路的情况
mysql> select true or null,false or null;
+--------------+---------------+
| true or null | false or null |
+--------------+---------------+
|            1 |          NULL |
+--------------+---------------+
1 row in set (0.00 sec)
```

---

## null 与 exists
构造一个 null 在子查询中的例子。
```sql
-- person 表的内容
mysql> select * from person;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | Tim   |   16 |
|  2 | Jerry |   20 |
|  3 | Tom   | NULL |
+----+-------+------+
3 rows in set (0.00 sec)

-- numbers 表的内容
mysql> select * from numbers;
+------+
| x    |
+------+
|    1 |
|    2 |
| NULL |
+------+
3 rows in set (0.00 sec)

```
查询 person.id 没有出现在 numbers.x 中的那些行(也就是这个例子中的 id = 3)。
```sql
-- 想像中的 person.id = 3 的那一行并没有出现，
mysql> select * from person where id not in (select x from numbers);
Empty set (0.01 sec)
```
想像中的 person.id = 3 的那一行并没有出现，详情可以看 [null 导致空结果](/blogs/119435262) ，上文给的建议是在数据库中尽可能的定义`not null default xxx` 来回避问题，本文就打算正面刚这个问题。

---

用 not exists 查询出 person.id 不存在于 numbers.x 中的那些行。
```sql
mysql> select *
    -> from person as a
    -> where not exists
    ->     (select x
    ->      from numbers as b
    ->      where b.x = a.id);  
+----+------+------+
| id | name | age  |
+----+------+------+
|  3 | Tom  | NULL |
+----+------+------+
1 row in set (0.00 sec)
```
exists 关注的是子查询有没有结果，至于结果集里是什么它不关心的，也就是说结果集可以是一个 null 的集合。
```sql
mysql> select *
    -> from person as a
    -> where not exists
    ->     (select null    -- 看这里子查询是 select null 哦。
    ->      from numbers as b
    ->      where b.x = a.id); 
+----+------+------+
| id | name | age  |
+----+------+------+
|  3 | Tom  | NULL |
+----+------+------+
1 row in set (0.00 sec)
```

---

## 结论
exists 之所以可以无视 null 是因为它检查的是集合是不是空集，而不管集合的元素是不是 null 。如果用的是 MySQL 数据库，exists 用不了半连接优化，所以相比 in 有可能会有一些性能上的问题。

---