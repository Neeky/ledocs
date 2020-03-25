## 背景
周末在自己的测试环境安装了一套 MGR 集群，当时是为了测试 dbm-agent-0.9.2 与 MySQL-8.0.18 有没有兼容性问题；

![mgr-error](static/2020-10/mgr-error.png)

google-adsense

---

## 故障重现
可以通过如下的 SQL 重现故障
```sql
mysql> create table t(x int) ;

mysql> insert into t(x) values(123);
ERROR 3098 (HY000): The table does not comply with the requirements by an external plugin.

-- 但是整个集群的状态是正常的

mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST       | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 02201545-6372-11ea-b92d-000c2956669d | mgr192_168_23_201 |        3306 | ONLINE       | SECONDARY   | 8.0.18         |
| group_replication_applier | 061dae8a-6372-11ea-88d7-000c29359c1c | mgr192_168_23_202 |        3306 | ONLINE       | SECONDARY   | 8.0.18         |
| group_replication_applier | eb4f421b-6371-11ea-87a2-000c29bfb653 | mgr192_168_23_200 |        3306 | ONLINE       | PRIMARY     | 8.0.18         |
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
```

---

## 查看错误日志
查看错误日志后发现，错误是原因是`mgr`要求表要有主键
```sql
2020-03-07T16:28:13.225051+08:00 36 [ERROR] [MY-011542] [Repl] Plugin group_replication reported: 'Table t does not have any PRIMARY KEY. This is not compatible with Group Replication.'
```

---

## 解决办法
原因明确了，那么解决办法就好找了，直接给表加上一个主键就行了
```sql
mysql> create table t(id int not null auto_increment primary key,v int);
Query OK, 0 rows affected (0.07 sec)

mysql> insert into t(v) values(100);
Query OK, 1 row affected (0.02 sec)

mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST       | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 02201545-6372-11ea-b92d-000c2956669d | mgr192_168_23_201 |        3306 | ONLINE       | SECONDARY   | 8.0.18         |
| group_replication_applier | 061dae8a-6372-11ea-88d7-000c29359c1c | mgr192_168_23_202 |        3306 | ONLINE       | SECONDARY   | 8.0.18         |
| group_replication_applier | eb4f421b-6371-11ea-87a2-000c29bfb653 | mgr192_168_23_200 |        3306 | ONLINE       | PRIMARY     | 8.0.18         |
+---------------------------+--------------------------------------+-------------------+-------------+--------------+-------------+----------------+
3 rows in set (0.00 sec)
```

---