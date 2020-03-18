## 背景
有同事反馈他创建表的时候报错了，表结构大致如下。
```sql
create table t(
    id int not null auto_increment,
    x int,
    y int,
    primary key(x,id)
) engine = innodb;
ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key
```
![Incorrect-table-definition](static/2020-12/Incorrect-table-definition.png)

---

## 原因
innodb 引擎有一个非常有意思的限制，当一个列被定义成 auto_increment 的时候，这个列要么独自作为主键；如果是想用联合索引做主键那么这个列一定要放到联合索引的第一列。

---

## 解决方案
知道问题出在哪里解决起来就方便了，针对这个问题我们要只把联合索引的位置换一下就行了。
```sql
create table t(
    id int not null auto_increment,
    x int,
    y int,
    primary key(id,x)  -- 把联合索引由(x,id) 调整成了 (id,x)
) engine = innodb;
Query OK, 0 rows affected (0.01 sec)
```
还有一种不推荐的方案，那就是直接把 Innodb 引擎给换掉，不过这都 2020 年了，还是不要再使用 MyISAM 的好。
```sql
create table t(
    id int not null auto_increment,
    x int,
    y int,
    primary key(x,id)
) engine = MYISAM; -- 直接把引擎给换成了 MYISAM 
Query OK, 0 rows affected (0.01 sec)
```

---