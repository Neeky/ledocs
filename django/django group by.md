## 概要
如何用 django 自带的 ORM 框架来完成 SQL 中的 group by 操作呢？

![group-by](static/2020-26/sqlpy-group-by.jpg)

google-adsense

---


## 环境介绍
foo 这个应用定义了如下模型。
```python
class PersonModel(models.Model):
    name = models.CharField('姓名', max_length=64)
    age = models.PositiveIntegerField('年龄')

    def __str__(self):
        return f"{self.name} - {self.age}"
```
数据库层面看到的表结构如下。
```sql
CREATE TABLE `foo_personmodel` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(64) COLLATE utf8mb4_general_ci NOT NULL,
  `age` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB
```
表中的数据如下。
```sql
mysql> select * from foo_personmodel;
+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | tim   |  16 |
|  2 | jerry |  27 |
|  3 | marry |  18 |
|  4 | tim   |  20 |
|  5 | alis  |  17 |
+----+-------+-----+
5 rows in set (0.00 sec)
```
---

## 全表聚合
一个常见的需求，查询一下表中有多少行数据。

SQL 写法。
```sql
mysql> select count(id) as counts from foo_personmodel;
+--------+
| counts |
+--------+
|      5 |
+--------+
1 row in set (0.00 sec)
```
ORM 实现。
```python
from django.db.models import Count
from apps.foo.models import PersonModel

PersonModel.objects.aggregate(counts = Count(id))

{'counts': 5}
```
注意 aggregate 返回的不再是 queryset 而是一个字典。

---

## values 方法
要查询出每一行 name 列的取值。

SQL 写法。
```sql
mysql> select name from foo_personmodel;
+-------+
| name  |
+-------+
| tim   |
| jerry |
| marry |
| tim   |
| alis  |
+-------+
5 rows in set (0.00 sec)
```
ORM 实现。
```python
PersonModel.objects.values("name")
<QuerySet [{'name': 'tim'}, {'name': 'jerry'}, {'name': 'marry'}, {'name': 'tim'}, {'name': 'alis'}]>
```
django 会返回一个 queryset 并且不会去重。

---


## 分组聚合
查询同一个名字在表中出现了多少次。

SQL 写法。
```sql
mysql> select name,count(id) as counts from foo_personmodel group by name;
+-------+--------+
| name  | counts |
+-------+--------+
| tim   |      2 |
| jerry |      1 |
| marry |      1 |
| alis  |      1 |
+-------+--------+
4 rows in set (0.00 sec)
```

ORM 写法。
```python
PersonModel.objects.values("name").annotate(counts=Count(id))
<QuerySet [{'name': 'tim', 'counts': 2}, {'name': 'jerry', 'counts': 1}, {'name': 'marry', 'counts': 1}, {'name': 'alis', 'counts': 1}]>
```
神奇的事情发生了，当 values 和 annotate 一起使用的时候，values 就承担起了 group by 的角色。并且自动去掉了重项!

---

## 结论
django 中并没有为 group by 设置单独的方法，而是通过 values + annotate 的组合来实现的。

---