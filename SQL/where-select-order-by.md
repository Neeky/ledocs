## 背景
SQL 语句比较特别，不是所有写在前面的都会先得到执行，如下面这一段。
```sql
select name as n,
       age as a
from person
where a = 19;

ERROR 1054 (42S22): Unknown column 'a' in 'where clause'
```

![select-ordery-by](static/2020-14/select-order-by.png)

google-adsense

---

## 原因
对于简单的 select 语句执行顺序大致上是这样的。
```bash
1、FROM
2、ON
3、JOIN
4、WHERE
5、GROUP BY
6、WITH CUBE 或 WITH ROLLUP
7、HAVING
8、SELECT
9、DISTINCT
10、ORDER BY
```
可以看到 `where` 在 `select` 之前执行，所以 'a' 这个别名在 'where' 执行的时候还不存在，进而报错了。

---

## 怎么绕过
可以看到只要让 where 执行的时候 'a' 这个名字存在就能解决问题了，于是我们可以通过子查询把起别名的这件事前置。
```sql
select * from (select name as n ,age as a from person) as t where t.a = 19;
+-------+------+
| n     | a    |
+-------+------+
| Jerry |   19 |
+-------+------+
1 row in set (0.00 sec)
```

虽然从语法上绕过了这个问题，但是还是不推荐这么写。

---

## order-by 
由于 order by 在 select 之后执行，所以 order by 中可以指定 select 起的别名。
```sql
select name as n,age as a from person order by a desc;
+-------+------+
| n     | a    |
+-------+------+
| Jerry |   19 |
| Tim   |   16 |
| Marry |   11 |
+-------+------+
3 rows in set (0.00 sec)
```

---



