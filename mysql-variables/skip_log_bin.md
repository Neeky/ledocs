## 背景
MySQL-8.0 版本下增加了一个专门的参数 `skip_log_bin` 来控制是否启用 `binlog`；相比之前的处理方法有明显的不同。

![skip-log-bin](static/2020-12/skip-log-bin.png)

google-adsense

---

## MySQL-8.0之前的版禁用binlog
MySQL-8.0 之前的版本只要注释掉 `binlog` 相关的配置项 MySQL 就会自己关闭 `binlog` 相关的功能了。
```init
[mysqld]
#log_bin = mysql-bin
```

---

## MySQL-8.0版本禁用binlog
MySQL-8.0 专门加了一个 skip_log_bin 的参数，专门用来做这个事，在 MySQL-8.0.19 版本下这个参数要出现在 `log_bin` 参数之后才有效果。
```ini
[mysqld]
log_bin      = mysql-bin
skip_log_bin = ON       # 注意 skip_log_bin 要放在 log_bin 之后才能生效
```
另一个要注意的事`skip_log_bin`参数是不能通过 `show global variables` 语句查看的。
```bash
mysql -uroot -pxxxx -h127.0.0.1 -P3306 -e"show global variables like '%log_bin%';"
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin                         | OFF   |
| log_bin_basename                |       |
| log_bin_index                   |       |
| log_bin_trust_function_creators | ON    |
| log_bin_use_v1_row_events       | OFF   |
+---------------------------------+-------+
```

---

