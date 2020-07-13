## 背景
我要把一些数据从 MySQL-5.6 的数据库迁移到 MySQL-5.7 ，并且我只想迁移业务数据，系统数据我并不想迁移。也就是说像 mysql,information_schema,performance_schema 这些我都不要。

mysqldump 是单线程的不知道要搞到什么时候才能完成，mysqlbackup 又收费，xtrabackup 备份完成之后还要升级 MySQL 才能用，最后我还是选择了 mydumper + myloader 来解决问题。原因有两个 1、mydumper 支持多线程备份 2、mydumper 支持正则表达式过滤出自己需要的库表。

mydumper 命令来说只有满足 --regex 正则表式指定的库表才会被导出，这个功能非常好的满足了我们需求。

![sqlpy](static/2020-29/sqlpy-dumper-regex.jpg)

---

## 数据源
数据源是一个 MySQL-5.6 的数据库，其内容如下。
```sql
mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 5.6.46    |
+-----------+
1 row in set (0.00 sec)

mysql> show databases;                                                                                            
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| tempdb             |
| test               |
+--------------------+
5 rows in set (0.00 sec)

-- tempdb.hello 是一个用户自定义函数
mysql> select tempdb.hello() as rows;
+------+
| rows |
+------+
|    5 |
+------+
1 row in set (0.00 sec)
```

google-adsense

---

## 数据目标
数据目标是一个 MySQL-5.7 的空实例。

---

## 导出
用 mydumper 导出 MySQL-5.6 中的数据，重要的是用正则表达式反选系统库之外的数据库。
```bash
mydumper -h 127.0.0.1 -u root -P 3306 --regex='^(?!mysql\.|information_schema\.|performance_schema\.|test\.)' \
--events --triggers --routines --outputdir=/tmp/mysql-3306
```
---


## 导入
用 myloader 把数据从  /tmp/3306 导到 MySQL-5.7。
```bash
myloader -h 127.0.0.1 -P 3307 -u root -d /tmp/u3306/
```
google-adsense

---

## 检查
由于我并没有导入系统库，所以像 function,procedure,view 这些与有用户有关的对象都会有问题。
```sql
mysql> show databases;                                                                                            
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| tempdb             |
+--------------------+
5 rows in set (0.00 sec)

mysql> select tempdb.hello();                                                                                     
ERROR 1449 (HY000): The user specified as a definer ('superdev'@'127.0.0.1') does not exist
```
可以看到程序报 'superdev'@'127.0.0.1' 用户不存在，解决的办法就是把这个用给建上。

---

## 补用户
目前看到的就只有这一个，不知道这种迁移方式还有没有其它的坑。
```sql
create user 'superdev'@'127.0.0.1' identified by 'xxxxxx';
grant all on *.* to 'superdev'@'127.0.0.1';

mysql> select tempdb.hello();                                                                                     
+----------------+
| tempdb.hello() |
+----------------+
|              5 |
+----------------+
1 row in set (0.00 sec)
```

---