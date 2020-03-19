## Distributed Recovery 
MGR 中把一个成功从`start group_replication` 开始到这个成员的状态为 `online`的这个恢复过程就称之为 `Distributed Recovery`

---

## Distributed Recovery的过程
第一步：如果成员之前就在这个组中，那么它的本地可能还保留有一些并没有被应用的`relay-log` MySQL 会先把这一部分日志应用完成。

第二步：连接到`donor`异步的同步 binlog 到本地并应用，当应用完成之后就广播说自己`online`了；事实上第二步是比较复杂的下面会专门展开说一下。

---


## DR0阶段
假设刚开始里有三个数据库结点，整个集群的状态如下，所有的结点都处于同一个状态(都应用了 20 个事务)。
![gr-recovery-1.png](static/2020-12/gr-recovery-1.png)

---

## DR1阶段
当有新的成员想加入到集群时，MGR 会向 binlog 中写入一个`view change`事件。
![gr-recovery-2.png](static/2020-12/gr-recovery-2.png)

这个事件是用来做标记的，当新加入的成员应用到这个事件时说明它和其它成员之前的差距已经非常小了；另外当新加入成员把`view change`件事后的其它事件(事务)都应用完成之后就会向集群广播说自己`online`了。

---

## DR2阶段
新加入成员开始侯追日志
![gr-recovery-3.png](static/2020-12/gr-recovery-3.png)

---

## DR3阶段
新加入成员应用完成`view change`事件
![gr-recovery-4.png](static/2020-12/gr-recovery-4.png)

---

## DR4阶段 
当新加入成员把 `view change`事件之后的其它事件应用完成之后就广播自己为`online`状态
![gr-recovery-5.png](static/2020-12/gr-recovery-5.png)

---

## view-change的重要性
`view chnage` 在分布式恢复的这个过程中有着非常重要的作用。 1、新成员要加入集群定入`view change`相当于是做了一个这个时间点的快照，可以看成是那个时间点上的一个全备。
2、对于新加入成员来说，当应用完这个`view change`事务的时候它就知道自己离最新的状态已经非常近了，所以当自己应该完成所有缓存下来的事件后就应该马上广播自己`online`的信号。

---


