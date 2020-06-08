## offline_mode 要解决的问题

当一个 MySQL 服务要下线的时候，之前的做法是直接把 MySQL 服务给关了，像下面这个样。
```bash
systemctl stop mysqld-3306
```

但是这样的做法有一个问题，比如突然业务发现这个库不能下线，业务的某一个流程会用到它，那我们又要启动 MySQL 服务。当数据量比较大的情况下，整个关闭重启的过程可能会很久。

而 offline_mode 为我们提供了一个更加轻巧的选择。

![sqlpy](static/2020-22/sqlpy-0608.jpg)

---


## offline_mode 的内部操作
当把参数 offline_mode 设置为 ON 时，MySQL-Server 会主动断开所有非特权连接(没有 super 域 connection_admin)，并且禁止他们再次连接。

google-adsense

---

## 实验
通过实验观察 offline_mode 对非特权连接的影响。

1、业务用户 appuser 正常的连接着数据库。
```sql
mysql> show grants;
+-----------------------------------------------------+
| Grants for appuser@127.0.0.1                        |
+-----------------------------------------------------+
| GRANT USAGE ON *.* TO `appuser`@`127.0.0.1`         |
| GRANT SELECT ON `tempdb`.* TO `appuser`@`127.0.0.1` |
+-----------------------------------------------------+
2 rows in set (0.00 sec)
```
---

2、root 用户把 MySQL-Server 离线。
```sql
mysql> set @@global.offline_mode=ON;                                                             
Query OK, 0 rows affected (0.00 sec)
```

---

3、当服务端为离线状态时会断开所有的非特权连接，所以 appuser 执行下一行命令时会报错，并且下一次连接也会失败。
```sql
-- 连接被断开了
mysql> show grants;                                                                              
ERROR 2013 (HY000): Lost connection to MySQL server during query

-- 再次连接会报错
mysql> show grants;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
ERROR 3032 (HY000): The server is currently in offline mode
ERROR: 
Can't connect to the server

```

这种情况下如果想要服务可以访问，只要把 offline_mode 设置为 OFF 就行了。

---