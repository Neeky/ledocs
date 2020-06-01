## 背景
有些情况下我们对 SQL 的时效性要求并不高，比如导出每日新增用户报表；还有一些时效性要求非常高的任务，比如注册。由于资源有限，我们要这两件事在同一个 MySQL 实例上完成(是的就是这么穷，一个 Slave 都没有)；

之前的解决方案是在业务压力低估时(通常是凌晨之后)执行导出报表的工作，MySQL 8 提出了新的解决方案资源组(Resource Groups)。

---

## Resource Groups 原理
MySQL 定义了一个资源组的概念，然后让单个连接或单条 SQL 语句隶属于某一个组，这样单个连接或单条 SQL 语句所能占用的资源总量就被这个组的资源总量限定住了。

资源组对资源的定义用白话来讲差不多就是 “把第一个到第四个 cpu 核心分配给资源组A，把第五个到第八个 cpu 核心分配给资源组B”，也就是说资源组是一个单纯的逻辑概念，同一个物理核心可以同时隶属于多个资源组。资源组之间通过优先级来控制谁有权使用资源。

看一下 MySQL 默认的资源组。
```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.RESOURCE_GROUPS ;    
+---------------------+---------------------+------------------------+--------------------+-----------------+
| RESOURCE_GROUP_NAME | RESOURCE_GROUP_TYPE | RESOURCE_GROUP_ENABLED | VCPU_IDS           | THREAD_PRIORITY |
+---------------------+---------------------+------------------------+--------------------+-----------------+
| USR_default         | USER                |                      1 | 0x302D31           |               0 |
| SYS_default         | SYSTEM              |                      1 | 0x302D31           |               0 |
+---------------------+---------------------+------------------------+--------------------+-----------------+
2 rows in set (0.00 sec
```

---


## 创建资源组
`create resource group` 语句可以用来创建资源组，在使用之前先确认一下主机核心数量。
```bash
cat /proc/cpuinfo  | grep processor
processor       : 0                                                                              
processor       : 1
```
可以看我是一个双核心的主机，现在分配一个核心给跑批用。
```sql
mysql> create resource group batch
    -> type = user
    -> vcpu = 1
    -> THREAD_PRIORITY = 15;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```
验证是是否创建成功。
```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.RESOURCE_GROUPS ;                                        
+---------------------+---------------------+------------------------+--------------------+-----------------+
| RESOURCE_GROUP_NAME | RESOURCE_GROUP_TYPE | RESOURCE_GROUP_ENABLED | VCPU_IDS           | THREAD_PRIORITY |
+---------------------+---------------------+------------------------+--------------------+-----------------+
| USR_default         | USER                |                      1 | 0x302D31           |               0 |
| SYS_default         | SYSTEM              |                      1 | 0x302D31           |               0 |
| batch               | USER                |                      1 | 0x31               |              15 |
+---------------------+---------------------+------------------------+--------------------+-----------------+
3 rows in set (0.01 sec)
```
如果在你的环境上看到 batch 组的 THREAD_PRIORITY == 0 应该是你的 MySQL 还没用启用 Resource Groups 功能，后面会讲到这个功能怎么启用。

google-adsense

---

## 使用资源组
总的来讲使用资源组有三种方式、分别是分当前线程(连接)指定、为特定的 SQL 语句指定、为其它线程指定。

1、指定当前连接使用哪个资源组。
```sql
mysql> set resource group batch;                                                                 
Query OK, 0 rows affected (0.00 sec)

mysql> select 'world' as hello;                                                                  
+-------+
| hello |
+-------+
| world |
+-------+
1 row in set (0.00 sec)
```

2、指定给定的 SQL 使用某个资源组，这个就要使用注释语法了。
```sql
mysql> select /*+ resource_group(batch) */ 'world' as hello;
+-------+
| hello |
+-------+
| world |
+-------+
1 row in set (0.00 sec
```
3、为某个线程指定资源组。
```sql
mysql> show processlist;                                                                         
+----+---------+-----------------+--------+---------+------+----------+------------------+
| Id | User    | Host            | db     | Command | Time | State    | Info             |
+----+---------+-----------------+--------+---------+------+----------+------------------+
|  7 | monitor | 127.0.0.1:36610 | NULL   | Sleep   |    2 |          | NULL             |
|  8 | root    | 127.0.0.1:36612 | NULL   | Sleep   |   94 |          | NULL             |
| 36 | root    | 127.0.0.1:36668 | tempdb | Query   |    0 | starting | show processlist |
+----+---------+-----------------+--------+---------+------+----------+------------------+
3 rows in set (0.00 sec)
```
现在让 7 这个会话对应的连接使用 batch 组
```sql
mysql> select thread_id,resource_group from performance_schema.threads where processlist_id=7 ;  
+-----------+----------------+
| thread_id | resource_group |
+-----------+----------------+
|        48 | USR_default    |
+-----------+----------------+
1 row in set (0.00 sec)

mysql> set resource group batch for 48;                                                          
Query OK, 0 rows affected (0.00 sec)

mysql> select thread_id,resource_group from performance_schema.threads where processlist_id=7 ;
+-----------+----------------+
| thread_id | resource_group |
+-----------+----------------+
|        48 | batch          |
+-----------+----------------+
1 row in set (0.00 sec)
```

---


## 删除资源组
删除资源组使用的语句是 `drop resource group`。
```sql
mysql> drop resource group batch;
ERROR 3656 (HY000): Resource group batch is busy.
```
如果有线程隶属于给定的资源组，那么资源组就不能 drop 。
```sql
kill 7;

mysql> show processlist;                                                                         
+----+---------+-----------------+--------+---------+------+----------+------------------+       
| Id | User    | Host            | db     | Command | Time | State    | Info             |
+----+---------+-----------------+--------+---------+------+----------+------------------+
|  8 | root    | 127.0.0.1:36612 | NULL   | Query   |    0 | starting | show processlist |
| 36 | root    | 127.0.0.1:36668 | tempdb | Sleep   |   71 |          | NULL             |
| 52 | monitor | 127.0.0.1:36700 | NULL   | Sleep   |    2 |          | NULL             |
+----+---------+-----------------+--------+---------+------+----------+------------------+
3 rows in set (0.00 sec)

mysql> drop resource group batch;
ERROR 3656 (HY000): Resource group batch is busy.
mysql> select thread_id,resource_group from performance_schema.threads where resource_group='batch';
+-----------+----------------+
| thread_id | resource_group |
+-----------+----------------+
|        49 | batch          |
+-----------+----------------+
1 row in set (0.00 sec)
```
kill 了也没有用，当连接重新连接之后还是去了那个组，看来解铃还须系铃人，用 set resource group 把线程移走吧。
```sql
mysql> set resource group USR_default for 49;
Query OK, 0 rows affected (0.00 sec)

mysql> drop resource group batch;
Query OK, 0 rows affected (0.00 sec)
```

---


## 启用资源组
想启用资源组要配置 systemd 的参数并从启 MySQL，关键参数就下面这一个。
```ini
[Service]
AmbientCapabilities=CAP_SYS_NICE
```
完整的参数如下。
```ini
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
AmbientCapabilities=CAP_SYS_NICE
User=mysql3306
Group=mysql
ExecStart=/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysqld --defaults-file=/etc/my-3306.cnf
LimitNOFILE = 102400
Environment=MYSQLD_PARENT_PID=1
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
```
配置好之后重新启动 MySQL 就行了。

---

## 官方文档

更多 [Resource Group](https://dev.mysql.com/doc/refman/8.0/en/resource-groups.html) 的信息还是看官方文档 。

---