## 背景
在访问 [www.sqlpy.com](https://sqlpy.com) 报 http 500 错误，难道 sqlpy.com 宕机了？我不太相信它会宕机；我又刷新了一下浏览器发现一切正常。

---

## 查看 django 日志
django 的日志中记录着如下内容。
```sql
2020-04-15 14:09:36,886 ERROR django.request Internal Server Error: /markets/golds/
Traceback (most recent call last):
  File "/usr/local/python-3.8.2/lib/python3.8/site-packages/django/db/backends/utils.py", line 86, in _execute
    return self.cursor.execute(sql, params)
  File "/usr/local/python-3.8.2/lib/python3.8/site-packages/django/db/backends/mysql/base.py", line 74, in execute
    return self.cursor.execute(query, args)
  File "/usr/local/python-3.8.2/lib/python3.8/site-packages/MySQLdb/cursors.py", line 209, in execute
    res = self._query(query)
  File "/usr/local/python-3.8.2/lib/python3.8/site-packages/MySQLdb/cursors.py", line 315, in _query
    db.query(q)
  File "/usr/local/python-3.8.2/lib/python3.8/site-packages/MySQLdb/connections.py", line 239, in query
    _mysql.connection.query(self, query)
MySQLdb._exceptions.OperationalError: (2006, 'MySQL server has gone away')
```

![gone-away](static/2020-16/gone-away.png)

google-adsense

---

## 分析 django 日志
从日志中可以看出是 `MySQL server has gone away` 这个错误，也就是应用程序到 MySQL-Server 的连接被断开了。这个错误有两个常见的场景。

第一个场景是查询的数据太多越过了最大包的限制。
```sql
mysql> show global variables like 'max_allowed_packet';
+--------------------+------------+
| Variable_name      | Value      |
+--------------------+------------+
| max_allowed_packet | 1073741824 |
+--------------------+------------+
1 row in set (0.00 sec)

mysql> select 1073741824 /1024/1024/1024;
+----------------------------+
| 1073741824 /1024/1024/1024 |
+----------------------------+
|             1.000000000000 |
+----------------------------+
1 row in set (0.00 sec)
```
可以看到参数 `max_allowed_packet` 已经是 1G 了，而我单次查询的结果集顶多几 k，就此排除了第一种可能。

---

第二个场景是连接被服务端主动断开了，如果一个连接长时间没有向服务端发送任何 `SQL` ，服务端就会认为客户端出了问题，连接没有正常断开，这个时候服务端就会主动断开连接。也就是说断开这类长时间没有发来 SQL 的连接是对服务端的一种保护。 

想一下如果客户端只是有连续 3 秒钟没有给服务端发送 SQL 语句，这个时候关闭连接是不太合理的；那要等多久才关闭呢？MySQL 给了一个参数 `wait_timeout` 用来设置多久没有交互就算超时。
```sql
mysql> show global variables like 'wait_timeout';
+---------------+----------+
| Variable_name | Value    |
+---------------+----------+
| wait_timeout  | 28800    |
+---------------+----------+
1 row in set (0.00 sec)

mysql> select 28800/3600;
+------------+
| 28800/3600 |
+------------+
|     8.0000 |
+------------+
1 row in set (0.00 sec)
```
找到了，8 小时没有交互就超时断开连接，这下我大胆点直接把它调整到一年。
```python
# 计算一下一年有多少秒(大概)
In [1]: 1 * 12 * 31 * 24 * 60 * 60                                              
Out[1]: 32140800

In [2]: exit
```

```sql
mysql> set @@global.wait_timeout = 32140800;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show global variables like 'wait_timeout';
+---------------+----------+
| Variable_name | Value    |
+---------------+----------+
| wait_timeout  | 31536000 |
+---------------+----------+
1 row in set (0.00 sec)
```

---


## 总结
通常这种情况只要调大 `max_allowed_packet` 和 `wait_timeout` 都能解决。[官方文档](https://dev.mysql.com/doc/refman/8.0/en/gone-away.html)。

---