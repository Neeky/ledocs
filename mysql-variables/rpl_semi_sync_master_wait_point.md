## lossless
要讲清理 lossless 就是要搞懂 `rpl_semi_sync_master_wait_point` 参数的两个取值 `AFTER_SYNC` | `AFTER_COMMIT` 有什么区别。

MySQL(innodb) 是支持事务的，要支持事务那就要日志先行，也就是说一个事务可以提交的前提是这个事务对应的日志都已经刷新到磁盘。

另外本文假设 MySQL 处在双1模式，也就是说两个关于日志刷新的重要参数的取值如下。

```sql
mysql> show global variables like 'sync_binlog';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sync_binlog   | 1     |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show global variables like 'innodb_flush_log_at_trx_commit';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)
```

![sync_binlog](static/2020-14/sync_binlog.png)

---

## commit 是一个过程
commit 不是一个时间上的点，它是时间轴上的一条线段，也就是说它是一个过程。但是这个过程中有几个关键的时间点，按先后次序一次是 `sync`，`commit`，`return` 如下图所示。

![wait-poing](static/2020-14/wait-poing.png)

`sync` 之后 `commit` 之前的这段时间称之为 `AFTER_SYNC`

`commit` 之后 `return ` 之前的这段时间称之为 `AFTER_COMMIT`

---

## rpl_semi_sync_master_wait_point
这个参数用于设置 master 在逻辑上的那个位置等待 slave 已经接收完 binlog 的确认信息(ACK信号)；目前只有两个取值 AFTER_SYNC 和 AFTER_COMMIT。

---

## 等于 AFTER_SYNC
当 rpl_semi_sync_master_wait_point = AFTER_SYNC `master` 就会在 AFTER_SYNC 这个时间段内等待 `slave` 确认信号。如果这个时候 `master` 宕机了，那么
这个事务在 `master` 上是不会被提交的，因为它本身并不有走过 `commit` 这个关键时间点。

如果 `slave` 收到了这个事务的完整日志，那么这个事务会在 slave 上提交，如果没有接收完成，那么这个事务在 `slave` 上也不会完成提交。最终 `slave` 的事务数至少是等于`master`的事务数。

因为等待的时候，事务并不有在 `master` 上提交，所以其它会话并看不到事务的修改。

---

## 等于 AFTER_COMMIT
当 rpl_semi_sync_master_wait_point = AFTER_COMMIT `master` 就会在 AFTER_COMMIT 这个时间段内等待 `slave` 确认信号。

如果这个时候 `master` 宕机了，那么这个事务在 `master` 已经是提交成功的状态，只是由于在等 `slave` 的确认信号没有返回给 `client`而已。

如果 `slave` 收到了这个事务的完整日志，那么这个事务会在 slave 上提交，如果没有接收完成，那么这个事务在 `slave` 上就不会提交。最终 `slave` 的事务数至多是等于`master`的事务数。

因为等待的时候，事务已经有在 `master` 上提交，所以其它会话能看到事务的修改。


---


