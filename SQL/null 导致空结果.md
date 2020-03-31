## 概要
null 在 SQL 语言中的位置比较特别，它表示不知道，并且只能通过 is 来测试。
```sql
-- 不管是 = 还是 != 都会返回 null，想知道是不是 null 有用的只有 is
mysql> select null = null,null != null,null is null ,null is not null;
+-------------+--------------+--------------+------------------+
| null = null | null != null | null is null | null is not null |
+-------------+--------------+--------------+------------------+
|        NULL |         NULL |            1 |                0 |
+-------------+--------------+--------------+------------------+
1 row in set (0.00 sec)
```

![null-empty](static/2020-14/null-empty.png)

也就是说 SQL 语言中不是二值逻辑(true & false)，它是三值逻辑(true & false & null)，由于有 null 的存在好多新手都坑哭了。

google-adsense

---

## 问题
我们知道 select 语句中只有 where 条件测试为 true 的行可以返回，也就是说测试结果为 null 和 false 的行都不会返回。
```sql
mysql> select * from person;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | Tim   |   16 |
|  2 | Jerry |   20 |
|  3 | Tom   | NULL |
+----+-------+------+
3 rows in set (0.00 sec)

-- 查询 id 为 1 和 2 的行
mysql> select * from person where id in (1,2);
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | Tim   |   16 |
|  2 | Jerry |   20 |
+----+-------+------+
2 rows in set (0.00 sec)
```
如果我们想把 1,2 两行排除掉，并且一不小心多写了一个 null 看一下会怎么样。
```sql
mysql> select * from person 
            where id not in (1,2,null);
Empty set (0.00 sec)
```
由于 null 的出现使得结果集为空了。

---

## 原因
1,2 没有在结果集里，非常正常因为刚好被排除了(select 只返回 where 条件测试为 true 的行)
```sql
mysql> select 1 not in (1,2,null),2 not in (1,2,null);
+---------------------+---------------------+
| 1 not in (1,2,null) | 2 not in (1,2,null) |
+---------------------+---------------------+
|                   0 |                   0 |
+---------------------+---------------------+
1 row in set (0.00 sec)
```
3 为什么也没有返回呢？因为 3 的测试结果不为 true ，它的测试结果是 null，又因为只有 true 才返回，所以它就没有返回。
```sql
mysql> select 3 not in (1,2,null);
+---------------------+
| 3 not in (1,2,null) |
+---------------------+
|                NULL |
+---------------------+
1 row in set (0.00 sec)
```
---

## 更加隐藏的情况
如果是 not in (subquery) 基本上就完蛋了。
```sql
create table numbers(x int);
insert into numbers(x) values(1),(2),(null);
mysql> select * from numbers;
+------+
| x    |
+------+
|    1 |
|    2 |
| NULL |
+------+
3 rows in set (0.00 sec)

mysql> select * from person;
+----+-------+------+
| id | name  | age  |
+----+-------+------+
|  1 | Tim   |   16 |
|  2 | Jerry |   20 |
|  3 | Tom   | NULL |
+----+-------+------+
3 rows in set (0.00 sec)

-- not in (subquery) 就真的完蛋了。

mysql> select * from person
            where id not in(select x from numbers);
Empty set (0.00 sec)
```

---


## 最佳实践是什么
尽可能让所有的列在定义的时候就加上 `not null default xxx`，这样就把三值逻辑降到了二值逻辑。

---