## 概要
MySQL-8.0.x 终于把四大约束给配齐了，也就是说 主键约束、外键约束、唯一键约束、check约束 都有了。


---

## 主键约束
主键约束的实施有两个影响，1、物理层面数据行会按主键的次序来组织，2、主键是一个特殊的唯一约束，它在保证唯一的同时还会要求不能为`NULL`。
```sql
create table if not exists teacher(
    id int not null auto_increment primary key, -- 就算不指定 not null 只要指定了 primary key ，mysql 也会自己加上 not null。
    name varchar(32) default '' comment '名字',
    salary decimal(8,2) default 0.0 comment '薪水'
);

mysql> show create table teacher;
+---------+-------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                              |
+---------+-------------------------------------------------------------------------------------------+
| teacher | CREATE TABLE `teacher` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT '' COMMENT '名字',
  `salary` decimal(8,2) DEFAULT '0.00' COMMENT '薪水',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci                                   |
+---------+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
---

由于有主键约束的存在插入 null 值会报错。
```sql
mysql> insert into teacher(id,name,salary) values(null,'tom',30000);                             
ERROR 1048 (23000): Column 'id' cannot be null
```
本例而言，由于表定义时候指定了 auto_increment 这个就使得当 null 值被传入的时候使用自增值。
```sql
insert into teacher(id,name,salary) values(null,'tom',30000);

mysql> select * from teacher;                                                                    
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
+----+------+----------+
1 row in set (0.00 sec)
```
---

主键不允许重复，如果插入重复的值就会报错。
```sql
mysql> select * from teacher;                                                                    
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
+----+------+----------+
1 row in set (0.00 sec)

mysql> insert into teacher(id,name,salary) values(1,'tom',30000);
ERROR 1062 (23000): Duplicate entry '1' for key 'teacher.PRIMARY'
```

---

## 外建约束
外键用于在一张表中引用另一张表中的数据，如学生表有一个班主什么任的列，而这个列应该引用的`teacher`表中的行。
```sql
create table student(
    id int not null auto_increment primary key,
    teacher_id int not null comment '班主任',
    name varchar(32) comment '姓名',

    foreign key fk_teacher_id(teacher_id) references teacher(id) on update no action on delete no action
);

mysql> show create table student;                                                                
+---------+--------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                                                                                                                                                                                                                                                                   |
+---------+--------------------------------------------------------------------------------------------+
| student | CREATE TABLE `student` (
  `id` int NOT NULL AUTO_INCREMENT,
  `teacher_id` int NOT NULL COMMENT '班主任',
  `name` varchar(32) DEFAULT NULL COMMENT '姓名',
  PRIMARY KEY (`id`),
  KEY `fk_teacher_id` (`teacher_id`),
  CONSTRAINT `student_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `teacher` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci                                     |
+---------+--------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
当被引用的数据行在父表中不存在时会报错。
```sql
mysql> select * from teacher;                                                                    
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
+----+------+----------+
1 row in set (0.00 sec)

-- 由于不存在 teacher.id 不存在等于 2 的行，所以下面的 SQL 会报错

mysql> insert into student(teacher_id,name) values(2,'jerry');                                   
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`tempdb`.`student`, CONSTRAINT `student_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `teacher` (`id`))
```

---

## 唯一键约束
唯一性约束就和它的名字一样，用于保证列的唯一性。
```sql
mysql> alter table student add constraint uid_name unique key (name);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

```
唯一键用于防止重复数据的写入。
```sql
-- 第一次，jerry 还不存在，所以可以成功.
mysql> insert into student(teacher_id,name) values(1,'jerry');                                   
Query OK, 1 row affected (0.01 sec)

-- 第二次，由于 jerry 已经存在了，所以会报错.
mysql> insert into student(teacher_id,name) values(1,'jerry');
ERROR 1062 (23000): Duplicate entry 'jerry' for key 'student.uid_name'
```

---


## check 约束
我们上面对 `teacher` 表的定义存在着一个问题，就是 salary 列可以取负值。 
```sql
mysql> insert into teacher(name,salary) values('bob',-100);                                      
Query OK, 1 row affected (0.00 sec)

mysql> select * from teacher;                                                                    
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
|  2 | bob  |  -100.00 |
+----+------+----------+
2 rows in set (0.00 sec)
```

想想薪水是一个负数真是一个有意思的工作，下面打算用 check 约束把这个杜绝掉。

```sql
mysql> select * from teacher;                                                                    
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
|  2 | bob  |  -100.00 |
+----+------+----------+
2 rows in set (0.00 sec)

-- 要把不满足条件的行删除掉，不然添加 check 会失败。
mysql> delete from teacher where id =2;                                                          
Query OK, 1 row affected (0.00 sec)

-- 添加 check 约束
mysql> alter table teacher add constraint ck_salary check (salary >= 0);
Query OK, 1 row affected (0.04 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from teacher;
+----+------+----------+
| id | name | salary   |
+----+------+----------+
|  1 | tom  | 30000.00 |
+----+------+----------+
1 row in set (0.00 sec)
```
现在写入非法数据就会报错了。
```sql
mysql> insert into teacher(name,salary) values('bob',-100); 
ERROR 3819 (HY000): Check constraint 'ck_salary' is violated.
```
---