## 背景
两条查询返回的结果集的格式是一样的可以写成一条吗？这像下面这两条 SQL 。
```sql
select name,age from person;
+-------+------+
| name  | age  |
+-------+------+
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
+-------+------+
3 rows in set (0.00 sec)

select name,age from people;
+-------+------+
| name  | age  |
+-------+------+
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
| neeky |   16 |
+-------+------+
4 rows in set (0.00 sec)

-- 表结构如下
CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) COLLATE utf8mb4_general_ci NOT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB

CREATE TABLE `people` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) COLLATE utf8mb4_general_ci NOT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB
```

![union](static/2020-14/union.png)

google-adsense

---

## union all 
`union all` 的作用是直接合并结果集(也就是说它不会对结果进行去重)。
```sql
select name,age from  person 

union all 

select name,age from people;

+-------+------+
| name  | age  |
+-------+------+
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
| neeky |   16 |
+-------+------+
7 rows in set (0.00 sec)
```
---

## union 
`union` 会合并两个结果集，然后在支结果集进行排序和去重。
```sql

select name,age from  person 

union 

select name,age from people;

+-------+------+
| name  | age  |
+-------+------+
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
| neeky |   16 |
+-------+------+
4 rows in set (0.00 sec)
```

---

## 注意事项
由于 union 要对结果集进行排序去重，而这都是比较重的操作最好不要在数据库层面执行。也就是说推荐在任何情况下都优先使用 union all 如果要去重就在应用层做。

由于  union 操作的是集合而集合是无序的，也就是说不能用`order by`
```sql
select name,age from  person order by age  
union   
select name,age from people;

ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'union   select name,age from people' at line 1
```
但是 order by 加到最后是可以的，这个时候它是对最终的结果集进行排序。
```sql
select name,age from  person   
union   
select name,age from people 
order by age desc;

+-------+------+
| name  | age  |
+-------+------+
| Jerry |   20 |
| Tim   |   16 |
| neeky |   16 |
| Tom   | NULL |
+-------+------+
4 rows in set (0.00 sec)
```

---

