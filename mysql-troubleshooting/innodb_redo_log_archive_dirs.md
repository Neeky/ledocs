## 面对的问题
redo log 物理上是多个文件，但是逻辑上它们是一个环(循环列表)；为什么说逻辑上是一个环呢？在当前 redo log 文件写满之后就会向下一个 redo log 文件进行写入，而最后一个 redo log 文件的下一个文件是第一个。也就是说 redo log 文件被设计成可重用的。

像 xtrabackup ，mysqlbackup 这样的热备份工具，它们是会实时盯住(读取并复制) redo log 文件，如果在它们还没有读完文件中剩余日志的情况下，redo log 日志就被复用了，那么备份就会失败。

在 5.6 和 5.7 版本下，如果想解决上面备份失败的问题，可能通过增加 redo log 文件数量，或提高 redo log 文件的大小来解决，但是这个解决方案要求重启 MySQL。

---

## redo-log 归档
redo-log 归档，是官方为应对备份失败而提出的一个解决方案。解决方式也比较直接，既然问题的一个因素是 redo-log 重用，那么让 MySQL 向一个新和目录写入 redo-log 文件，并且这些文件不再重用，备份工具直接从新的地方读不就行了。

---


## 手工开启归档
第一步：创建用于保存归档的目录并设置好权限。
```bash
mkdir /redos/mysql/redos/3306
chown mysql3306:mysql /redos/mysql/redos/3306
chmod 700 /redos/mysql/redos/3306
```
---
第二步：配置 MySQL 使用指定的归档目录。
```sql
mysql> set @@global.innodb_redo_log_archive_dirs=":/redos/mysql/redos/3306";                     
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'innodb_redo_log_archive_dirs';
+------------------------------+--------------------------+
| Variable_name                | Value                    |
+------------------------------+--------------------------+
| innodb_redo_log_archive_dirs | :/redos/mysql/redos/3306 |
+------------------------------+--------------------------+
1 row in set (0.00 sec)
```
---

第三步：启动归档
```sql
mysql> do innodb_redo_log_archive_start('');
Query OK, 0 rows affected (0.52 sec)
```
启动完成之后归档目录下会生成新的 redo 日志。
```bash
ll -h /redos/mysql/redos/3306/
-r--r-----. 1 mysql3306 mysql 0 5月   2 14:15 archive.e82a2ddf-89f0-11ea-b6ad-000c29e0ca28.000001.log
```

google-adsense

---

## 写入数据观察 redo-log 的变化
```sql
mysql> create database tempdb;                                                                   
Query OK, 1 row affected (1.01 sec)

mysql> use tempdb ;                                                                              
Database changed
mysql> show tables;
Empty set (0.00 sec)

mysql> create table t(x int);                                                                    
Query OK, 0 rows affected (0.01 sec)

mysql> insert into t(x) values(100),(200),(300);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

可以看到 redo-log 的归档空间上涨了。

```bash
ll -h /redos/mysql/redos/3306/
-r--r-----. 1 mysql3306 mysql 28K 5月   2 14:18 archive.e82a2ddf-89f0-11ea-b6ad-000c29e0ca28.000001.log
```

---

## 关闭 redo-log 归档
```sql
mysql> do innodb_redo_log_archive_stop();                                                    
1 row in set (0.00 sec)
```

---


## 参考
更加详细的说明可以查看官方文档 [archive-redo-log](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html)。



---






