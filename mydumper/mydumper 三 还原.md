## myloader 
myloader 是 `mydumper` 软件包中用于还原数据库的命令，整体上看这个命令的选项不多，用起来应该不难。
```bash
myloader --help                                                             
Usage:
  myloader [OPTION?] multi-threaded MySQL loader

Help Options:
  -?, --help                        Show help options

Application Options:
  -d, --directory                   Directory of the dump to import
  -q, --queries-per-transaction     Number of queries per transaction, default 1000
  -o, --overwrite-tables            Drop tables if they already exist
  -B, --database                    An alternative database to restore into
  -s, --source-db                   Database to restore
  -e, --enable-binlog               Enable binary logging of the restore data
  -h, --host                        The host to connect to
  -u, --user                        Username with the necessary privileges
  -p, --password                    User password
  -a, --ask-password                Prompt For User password
  -P, --port                        TCP/IP port to connect to
  -S, --socket                      UNIX domain socket file to use for connection
  -t, --threads                     Number of threads to use, default 4
  -C, --compress-protocol           Use compression on the MySQL connection
  -V, --version                     Show the program version and exit
  -v, --verbose                     Verbosity of output, 0 = silent, 1 = errors, 2 = warnings, 3 = info, default 2
  --defaults-file                   Use a specific defaults file
```


---

## 删库
由于我们已经有了 `tempdb` 的备份，现在把 tempdb 删除掉，体验一下用 myloader 还原的效果怎样。
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
5 rows in set (0.02 sec)

mysql> drop database tempdb;
Query OK, 1 row affected (1.03 sec)
```

google-adsense

---

## 还原
用 myloader 还原数据库。
```bash
myloader --host=127.0.0.1 --port=3306 --user=root --password=dbma@0352 --directory=/tmp/3306/
```

---

## 检查
因为 tempdb 中有数据非常少，所以还原起来也非常快，下面验证一下还原的效果。
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

mysql> select * from tempdb.scores;
+----+--------------+-------+--------+-------------+
| id | student_name | math  | physis | total_score |
+----+--------------+-------+--------+-------------+
|  1 | tom          | 120.0 |  100.0 |       220.0 |
+----+--------------+-------+--------+-------------+
1 row in set (0.00 sec)
```

---