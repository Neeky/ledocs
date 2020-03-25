## 背景
社区反馈说 dbm-agent 安装 mysql 时会有 error 级别的日志产生，虽然不影响使用但是总给人一种不好的感觉。

![init-error](static/2020-11/init-error.png)

```log
2020-03-13T10:33:21.372003+08:00 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.18) initializing of server in progress as process 7602
2020-03-13T10:33:24.149142+08:00 5 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2020-03-13T10:33:24.954311+08:00 5 [ERROR] [MY-000061] [Server] 1146  Table 'mysql.backup_progress' doesn't exist.
2020-03-13T10:33:24.954806+08:00 0 [ERROR] [MY-013236] [Server] The designated data directory /database/mysql/data/3306/ is unusable. You can remove all files that the server added to it.
2020-03-13T10:33:24.954815+08:00 0 [ERROR] [MY-010119] [Server] Aborting
2020-03-13T10:33:26.571808+08:00 0 [System] [MY-010910] [Server] /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64/bin/mysqld: Shutdown complete (mysqld 8.0.18)  MySQL Community Server - GPL.
2020-03-13T10:33:26.926382+08:00 0 [System] [MY-010116] [Server] /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.18) starting as process 7696
2020-03-13T10:33:27.912444+08:00 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-03-13T10:33:27.914800+08:00 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/tmp' in the path is accessible to all OS users. Consider choosing a different directory.
2020-03-13T10:33:27.936489+08:00 0 [System] [MY-010931] [Server] /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64/bin/mysqld: ready for connections. Version: '8.0.18'  socket: '/tmp/mysql-3306.sock'  port: 3306  MySQL Community Server - GPL.
2020-03-13T10:33:27.936507+08:00 0 [System] [MY-013292] [Server] Admin interface ready for connections, address: '127.0.0.1'  port: 33062
2020-03-13T10:33:28.188198+08:00 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/tmp/mysqlx-33060.sock' bind-address: '::' port: 33060

```

---

## 日志分析
日志的第一段是由 `initialize-insecure` 产生的、也就是说这个时候并没有报错。
```log
2020-03-13T10:33:21.372003+08:00 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.18-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.18) initializing of server in progress as process 7602
2020-03-13T10:33:24.149142+08:00 5 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
```
日志的第二段是由 `init-file` 产生的，init-file 中的内容主要是创建用户和授权。
```log
2020-03-13T10:33:24.954311+08:00 5 [ERROR] [MY-000061] [Server] 1146  Table 'mysql.backup_progress' doesn't exist.
2020-03-13T10:33:24.954806+08:00 0 [ERROR] [MY-013236] [Server] The designated data directory /database/mysql/data/3306/ is unusable. You can remove all files that the server added to it.
2020-03-13T10:33:24.954815+08:00 0 [ERROR] [MY-010119] [Server] Aborting
```
原因给一个不存在的表授`alter`权限是会报错的、但是和 `create` 一起授予就不会报错。
```sql
mysql> grant alter on mysql.backup_progress to 'mysqlbackup'@'127.0.0.1';
ERROR 1146 (42S02): Table 'mysql.backup_progress' doesn't exist
-- 单独的 alter 是有问题的，但是和 create 一起是 OK 的
mysql> grant create,alter,insert, drop, update on mysql.backup_progress to 'mysqlbackup'@'127.0.0.1';
Query OK, 0 rows affected (0.01 sec)
```
init-file 的写法正好命中这个问题
```
grant create,insert, drop, update on mysql.backup_progress to 'mysqlbackup'@'127.0.0.1';
grant alter on mysql.backup_progress to 'mysqlbackup'@'127.0.0.1';
```
google-adsense

---

## 解决方案
把分开的两次授权合并到一次里
```sql
grant create,alter,insert, drop, update on mysql.backup_progress to 'mysqlbackup'@'127.0.0.1';
```
另外 dbm-agent-0.9.4 版本已经解决了这个问题
```log
2020-03-13T11:13:20.511458+08:00 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.19-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.19) initializing of server in progress as process 2468     
2020-03-13T11:13:22.854711+08:00 5 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.              
2020-03-13T11:13:25.702342+08:00 0 [System] [MY-010116] [Server] /usr/local/mysql-8.0.19-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.19) starting as process 2563                               
2020-03-13T11:13:26.262654+08:00 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.                                                                                           
2020-03-13T11:13:26.265959+08:00 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/tmp' in the path is accessible to all OS users. Consider choosing a different directory.                                                                                       
2020-03-13T11:13:26.291613+08:00 0 [System] [MY-010931] [Server] /usr/local/mysql-8.0.19-linux-glibc2.12-x86_64/bin/mysqld: ready for connections. Version: '8.0.19'  socket: '/tmp/mysql-3306.sock'  port: 3306  MySQL Community Server - GPL.                                                    
2020-03-13T11:13:26.291641+08:00 0 [System] [MY-013292] [Server] Admin interface ready for connections, address: '127.0.0.1'  port: 33062                                                         
2020-03-13T11:13:26.374435+08:00 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/tmp/mysqlx-33060.sock' bind-address: '::' port: 33060
```
