## null 值与 not null 混合排序
用一个例子来说明吧，有一个 person 表其它包含`name`和`age`两列，现在要按`age`的升序排列查询结果，但是要求把`age`等于 `null`的排到最后。
```sql
create table person(
    id int not null auto_increment primary key,
    name varchar(32) not null,
    age int);

insert into person(name,age) values('Tim',16),('Jerry',20),('Tom',null);

```
直接对 age 进行升序排列是解决不了的
```sql
select name,age from person order by age;
+-------+------+
| name  | age  |
+-------+------+
| Tom   | NULL |
| Tim   |   16 |
| Jerry |   20 |
+-------+------+
3 rows in set (0.00 sec)
```
![null-not-null](static/2020-27/null-not-null.jpg)

google-adsense


---

## 解析方案
第一步：由于 null 不好比较所以先把它转化为值。
```sql
select name,
       age,
       case
           when age is null then 1
           else 0
       end as age_is_null
from person ;

+-------+------+-------------+
| name  | age  | age_is_null |
+-------+------+-------------+
| Tim   |   16 |           0 |
| Jerry |   20 |           0 |
| Tom   | NULL |           1 |
+-------+------+-------------+
3 rows in set (0.00 sec
```
现在可以看出要想 age 等待 null 的列排在后面就是要按 age_is_null 的升序排列就行，由于结果又不能出现 age_is_null 所以直接加一个外层查询。

---

第二步：加一层外层查询
```sql
select name,
       age
from
  (select name,
          age,
          case
              when age is null then 1
              else 0
          end as age_is_null
   from person) as a
order by a.age_is_null,
         a.age;  
+-------+------+
| name  | age  |
+-------+------+
| Tim   |   16 |
| Jerry |   20 |
| Tom   | NULL |
+-------+------+
3 rows in set (0.00 sec)
```

---

## SQL格式化
可通过 [sql-format](https://sqlpy.com/onlinetools/sqlformat) 这个 web 工具完成 SQL 格式化。

---