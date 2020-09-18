## 背景

最近工作上的事情比较多，好长一段时间没有总结了。刚好昨天晚上 2 点多被告警电话打醒，上机器一看所有从库的复制都断了。

![sqlpy](static/2020-37/1595.jpg)

---


## 现象

1、当时的复制状态如下，可以看到 slave 的 io 线程报错了。

```sql
[ mysql ] >show slave status \G
*************************** 1. row ***************************
             Slave_IO_Running: No
            Slave_SQL_Running: Yes

                Last_IO_Errno: 1595
                Last_IO_Error: Relay log write failure: could not queue event from master

      Slave_SQL_Running_State: Reading event from the relay log
1 row in set (0.00 sec)

```

当时情况比较紧急我就直接把 io 线程重新启动了下，问题解决了。还手工并保留了现场，用于后面的分析。
```sql
[ mysql ] >start slave io_thread;

[ mysql ] >show slave status \G
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

---

## 分析
1、分析当时出问题时的 binlog 可以看到业务在执行一个 `load data` 的操作，导致整个 binlog 文件膨胀到了 5.3G 。
```bash
-rw-r-----. 1 mysql3306 mysql 5.3G 9月  16 02:31 mysql-bin.010004
```

现在问题就比较明确了，业务执行了一个大的事务，master 把整个的 binlog 传给 slave ；一方面这个事务比较大，master 要传好久，另一方面 master 向 slave 传心跳信息和传 binlog 用的是同一条 tcp 连接，这就导致了 slave 在好长一段时间之内没有收到 master 发来的心跳。

这种情况下 slave 就会认为 master 宕机了，主动的关闭(io-thread线程) tcp 连接 ，这样就有了刚才的告警。

---

## 解决方案
slave 等待 master 心跳的超时时间由 `slave_net_timeout` 这个参数控制，所以我们要在 slave 上把这个参数调大，并重启启动复制。
```sql
[ mysql ] >set @@global.slave_net_timeout=120 -- 前值是 30 我的千兆机器下面挂了多台 slave ，网卡流量有些不足。
[ mysql ] >stop slave;
[ mysql ] >start slave;

```
---

## 其它资源
1、腾讯的 txsql 团队向官方提了这个 bug [95418](https://bugs.mysql.com/bug.php?id=95418)，可能官方的网络环境比较好吧，所以没有复现出来。

2、[slave_net_timeout](https://dev.mysql.com/doc/refman/5.7/en/replication-options-replica.html#sysvar_slave_net_timeout) 参数的官方说明。

---






