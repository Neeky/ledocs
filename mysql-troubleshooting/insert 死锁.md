## insert 上锁步骤
insert语句上锁的大致过程如下：

1、在行所在的间隙上申请“意向插入锁”。2、申请所要插入行的“排他锁”。3、如果在第二步的时候引发了唯一键冲突，那么陷入冲突的事务，要把上锁的过程分两步，先申请行的“共享锁”，然后再申请“排他锁”； 如果有多个事物陷入冲突，那么他们一定都会申请到“共享锁”，然后在申请排他锁的相互等待(死锁)，这个时候 MySQL 会选择牺牲掉其中一些事务，让其中的一个完成。

google-adsense

---

## insert 死锁
按理论一步一步的复现 insert 死锁的场景。

|**会话一**|**会话二**|**会话三**|
|---------|---------|---------|
|create table t(x int primary key);| | |
|start transaction;|||
|insert into t(x) values(1024);|||
||insert into t(x) values(1024);|insert into t(x) values(1024);|
|rollback|commit successful|ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction|

---

## 死锁日志如下
`show engine innodb status` 输出中关于死锁的信息如下。

```sql
------------------------
LATEST DETECTED DEADLOCK
------------------------
2019-09-03 19:29:30 0x7f9ab00b4700
*** (1) TRANSACTION:
TRANSACTION 2759830, ACTIVE 9 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 13, OS thread handle 140302402717440, query id 1221 127.0.0.1 root update
insert into t(x) values(1024)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 16 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 2759830 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) TRANSACTION:
TRANSACTION 2759831, ACTIVE 6 sec inserting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 14, OS thread handle 140302355220224, query id 1222 127.0.0.1 root update
insert into t(x) values(1024)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 16 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 2759831 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 16 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 2759831 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** WE ROLL BACK TRANSACTION (2)
```

---

## MySQL 为什么要这么做

大前提是这样的，两个事务插入同一条记录，必定有一个要失败。死锁也是必定要有一个失败，那么何不把并发插入也合并到死锁的处理流程中去呢？想到就要做到，那把上行锁的过程拆一下，
先上“S 共享锁” 再上“X 排他锁”，由于“S”可以共享所以两个都能完成这一步，第二小要“X”所以两个事务就相互等待(死锁)。

---


## 解决方案
如果你在业务上还真遇到了上面的场景，然而 MySQL 死锁了。你又不想让它死锁，那怎么办呢？MySQL 不是把上锁的步骤拆开了吗？那我们把它合起来。

`select ... for update` 直接对行上排他锁，通过它我们可以把分开的两步合起来，`on duplicate key update` 当遇到冲突时会直接更新，不会再报错；结合这两大神器代码可以改成如下的形式。

|**会话一**|**会话二**|**会话三**|
|---------|---------|---------|
|create table t(x int primary key);| | |
|start transaction;|||
|select x from t for update; insert into t(x) values(1024) on duplicate key update x = 1024;|||
||select x from t for update; insert into t(x) values(1024) on duplicate key update x = 1024;|select x from t for update; insert into t(x) values(1024) on duplicate key update x = 1024;|
|rollback|commit successful|commit successful|


---

## 副作用

由于 `select for update` 是排他锁，所以并发性上会有些问题，建议与`read-committed`一起使用。

---

## 彩蛋
根据分析可以知道问题发生在行锁上，把隔离级别调整成 `READ-COMMITTED` 也只会影响 `gap` 锁，我以为这个隔离级别下不会再有插入意向锁(一种特别的 gap)了，没想到还有。

```sql
mysql> show variables like '%iso%';
+-----------------------+----------------+
| Variable_name         | Value          |
+-----------------------+----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+----------------+
1 row in set (0.00 sec)
```
---

执行同样的操作。

|**会话一**|**会话二**|**会话三**|
|---------|---------|---------|
|create table t(x int primary key);| | |
|start transaction;|||
|insert into t(x) values(1024);|||
||insert into t(x) values(1024);|insert into t(x) values(1024);|
|rollback|commit successful|ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction|

---

```sql
------------------------
LATEST DETECTED DEADLOCK
------------------------
2020-04-11 13:57:15 0x7ff1bccf1700
*** (1) TRANSACTION:
TRANSACTION 6946, ACTIVE 12 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 9, OS thread handle 140676621076224, query id 237 127.0.0.1 root update
insert into t(x) values(1024)

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 5 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 6946 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 6946 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;


*** (2) TRANSACTION:
TRANSACTION 6947, ACTIVE 9 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 13, OS thread handle 140676151592704, query id 258 127.0.0.1 root update
insert into t(x) values(1024)

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 5 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 6947 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5 page no 4 n bits 72 index PRIMARY of table `tempdb`.`t` trx id 6947 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

*** WE ROLL BACK TRANSACTION (2)
```

`READ-COMMITTED` 隔离级别下已经没有 `gap` 锁了，然而官方文档上说 `insert intention` 是 gap 的一种，所以 `insert intention` 应该不存在才对呀。

---
