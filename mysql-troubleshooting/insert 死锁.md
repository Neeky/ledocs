## insert 上锁步骤
insert语句上锁的大致过程如下：

1、在行所在的间隙上申请“意向插入锁”。2、申请所要插入行的“排他锁”。3、如果在第二步的时候引发了唯一键冲突，那么陷入冲突的事务，要把上锁的过程分两步，先申请行的“共享锁”，然后再申请“排他锁”； 如果有多个事物陷入冲突，那么他们一定都会申请到“共享锁”，然后在申请排他锁的相互等待(死锁)，这个时候 MySQL 会选择牺牲掉其中一些事务，让其中的一个完成。

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
