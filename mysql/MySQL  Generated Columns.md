## Generated Columns 要解决的问题
`Generated Columns` 就是在平衡存储开销和计算开销；假设有一个表用来保存学生的成绩，当学生查询自己在年级的排名时，应用程序从数据库中读取出各科的成绩，求和，再由高到低排序。

每一次查询都要求和，那为什么不把求先保存起来呢？这样后面要用到的时候就直接取。之前这个工作要在应用程序来实现，现在 MySQL 提供了原生的支持。

---

## 古老手法
在生成列还没有出来之前上面介绍的功能可以这样实现。
```sql
create table scores(
    id int not null auto_increment primary key,
    student_name varchar(32) not null comment '学生名',
    math decimal(4,1) not null comment '数学',
    physis decimal(4,1) not null comment '物理'
);

mysql> select student_name,math,physis,sum(math+physis) as total_score from scores group by id;  
+--------------+-------+--------+-------------+
| student_name | math  | physis | total_score |
+--------------+-------+--------+-------------+
| tom          | 120.0 |  100.0 |       220.0 |
+--------------+-------+--------+-------------+
1 row in set (0.00 sec)
```

google-adsense

---

## 生成列语法
生成列语法在数据类型后面加上了新的子句。
```sql
col_name data_type [GENERATED ALWAYS] AS (expr)
  [VIRTUAL | STORED] [NOT NULL | NULL]
  [UNIQUE [KEY]] [[PRIMARY] KEY]
  [COMMENT 'string']
```
语法规则上来看还是比较简单的，下面直接看例子。

---

## STORED 模式
stored 模式会实实在在的把计算后的结果保存下来，方便下次直接用；现在我们用 stored 模式解决上面的问题。
```sql
create table scores(
    id int not null auto_increment primary key,
    student_name varchar(32) not null comment '学生名',
    math decimal(4,1) not null comment '数学',
    physis decimal(4,1) not null comment '物理',
    total_score decimal(6,1) as (math + physis) stored not null  comment '总成绩'
);
Query OK, 0 rows affected (0.01 sec)

-- 插入数据
mysql> insert into scores(student_name,math,physis) values('tom',120,100);
Query OK, 1 row affected (0.01 sec)

-- 和直接就出来了
mysql> select * from scores;
+----+--------------+-------+--------+-------------+
| id | student_name | math  | physis | total_score |
+----+--------------+-------+--------+-------------+
|  1 | tom          | 120.0 |  100.0 |       220.0 |
+----+--------------+-------+--------+-------------+
1 row in set (0.00 sec)

-- 生成列是由 MySQL 自己维护的
mysql> update scores set math = 100 where id =1;                                                 
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

-- 看总成绩自己更新了哦
mysql> select * from scores;                                                                     
+----+--------------+-------+--------+-------------+
| id | student_name | math  | physis | total_score |
+----+--------------+-------+--------+-------------+
|  1 | tom          | 100.0 |  100.0 |       200.0 |
+----+--------------+-------+--------+-------------+
1 row in set (0.00 sec)
```

google-adsense

---

## VIRTUAL 模式
前面说到 stored 模式会实实在在的把计算过后的成绩保存起来，virtual 模式不会保存计算列，它只在读取的时候计算，所以这个解决方案节约了磁盘，但是会多用一些 cpu 。
```sql
create table scores(
    id int not null auto_increment primary key,
    student_name varchar(32) not null comment '学生名',
    math decimal(4,1) not null comment '数学',
    physis decimal(4,1) not null comment '物理',
    total_score decimal(6,1) as (math + physis) virtual not null  comment '总成绩'
);

mysql> insert into scores(student_name,math,physis) values('tom',120,100);
Query OK, 1 row affected (0.00 sec)

mysql> select * from scores;
+----+--------------+-------+--------+-------------+
| id | student_name | math  | physis | total_score |
+----+--------------+-------+--------+-------------+
|  1 | tom          | 120.0 |  100.0 |       220.0 |
+----+--------------+-------+--------+-------------+
1 row in set (0.00 sec)

```

---

## 总结

`Generated Columns` 有的人叫它“计算列”，又有的叫它“虚拟列”；但是不管怎样知道它解决什么问题，并且用好它还是不难的，更详细的可以看[官方文档](https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html)。

---


