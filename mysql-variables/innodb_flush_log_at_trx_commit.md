## innodb_flush_log_at_trx_commit
innodb_flush_log_at_trx_commit 用于平衡数据库性能与宕机后数据的可恢复性(安全性)；为了保证安全性每一笔写操作都要需要对应的日志被刷新到磁盘，然而 IO 操作是比较慢的，如果我们对安全性要求不高那大可不要每次 commit 前都要把日志刷新到磁盘。

---


## 1
当 innodb_flush_log_at_trx_commit = 1 每一次 commit 之前都会把日志刷新到磁盘；因为先写日志再 commit 所以它是宕机安全的；但是性能上就不怎么行了。
google-adsense

---


## 0
当 innodb_flush_log_at_trx_commit=0 每一秒都会执行 write + flush 操作；因为是每秒执行一次，所以有宕机丢失日志的风险；但是性能上就比 innodb_flush_log_at_trx_commit=1 的时候要好一些。

---

## 2

innodb_flush_log_at_trx_commit = 1 和 innodb_flush_log_at_trx_commit = 0 是两个极端，1 时要求 commit 之前完成 `write + flush` 操作， 0 时要求每秒一次`write + flush` 操作；1 会导致日志写的太频繁，0 时又写的太懒惰了每秒才一次。

innodb_flush_log_at_trx_commit = 2 时会在每个事务 commit `之后`执行一次 `write` ，这种情况下由于是 commit 之后触发 `write` 所以不怎么影响性能；光 `write` 不能保证日志被写入到磁盘的，它也会每秒执行一次`flush`操作。

---

## 总结
经验上看 innodb_flush_log_at_trx_commit = 1 时会比 innodb_flush_log_at_trx_commit = 2 和 innodb_flush_log_at_trx_commit = 0 在性能上差一个数量级

---

