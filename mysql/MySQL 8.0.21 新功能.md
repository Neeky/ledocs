## 背景
MySQL-8.0.21 在用户管理语句上添加了新的子句，现在可以给用户打上更“注释”，和更多的其它属性，使用方法也比较简单如下所示。

```sql
mysql> create user sqlpy@'127.0.0.1' identified by 'xxxxx' comment '应用程序账号';                                
Query OK, 0 rows affected (1.01 sec)

mysql> alter user sqlpy@'127.0.0.1' attribute '{"desc":"web site backend user."}';
Query OK, 0 rows affected (0.00 sec)
```
![sqlpy](static/2020-29/sqlpy-user.jpg)

---

## show-grants

通过 show grants 并不能查出 comment 和 attribute 中的内容。
```sql
mysql> show grants for sqlpy@'127.0.0.1';                                                                         
+-------------------------------------------+
| Grants for sqlpy@127.0.0.1                |
+-------------------------------------------+
| GRANT USAGE ON *.* TO `sqlpy`@`127.0.0.1` |
+-------------------------------------------+
1 row in set (0.00 sec)
```

---


## 数据保存到了哪里

8.0.21 在 mysql.user 表中添加了一个新的列 `user_attributes` 它是 json 类型的。
```sql
mysql> select user_attributes from mysql.user where user = 'sqlpy';                                               
+-----------------------------------------------------------------------------------+
| user_attributes                                                                   |
+-----------------------------------------------------------------------------------+
| {"metadata": {"desc": "web site backend user.", "comment": "应用程序账号"}}       |
+-----------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

---