## pt-online-schema-change 要解决的问题
DDL 语句的执行时间直接与表中的数据量正相关，我经过过一个 `alter table` 下去半个多小时才完成的；如果这个数据库有从库那么从库，那么从库的 SQL 线程也要执行这么久才行，这就会造成主从延时，应用程序在从库上查询不到最新的数据。虽然 MySQL-8.0 已经好多了，但是对于 MySQL-5.6 & MySQL-5.7 来说，如果表比较大使用 pt-online-schema-change 还是有必要的。

![sqlpy](static/2020-26/sqlpy-pt-osc.jpg)

---

## 使用 pt 执行 DDL 变更
假设 `tempdb.t` 表的结构和数据如下
```sql
mysql> show create table t;                                                                      
+---------------------------------------------------+
| Table | Create Table                                                                                                                                                                            |
+---------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int NOT NULL AUTO_INCREMENT,
  `v` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB |
+---------------------------------------------------+
1 row in set (0.00 sec)

mysql> select * from t;                                                                   
+----+------+
| id | v    |
+----+------+
|  3 |    3 |
+----+------+
1 rows in set (0.00 sec)
```

google-adsene

---

2、我们现在要给定加上一个 `int not null default 0` 列，直接 alter table 的话可以这样写。
```sql
alter table tempdb.t add column i int not null default 0;
```

因为我们这里是要介绍 pt-online-schema-change 所以就不用这面的 DDL 语句来做表结构和变更了。

---

3、用 pt-online-schema-change 执行 DDL 语句。
```bash
pt-online-schema-change --alter="add column i int not null default 0" --execute h=127.0.0.1,P=3306,D=tempdb,u=superdev,p=dbma@0352,t=t 

```

效果如下。

```sql
mysql> select * from t;                                                                          
+----+------+---+
| id | v    | i |
+----+------+---+
|  3 |    3 | 0 |
+----+------+---+
1 row in set (0.00 sec)
```
---


## pt-online-schema-change 的程序流程
第一步、检查要变更的表，的表结构。
```sql
SHOW CREATE TABLE `tempdb`.`t`;
```
第二步、创建同结构的新表。
```sql
CREATE TABLE `tempdb`.`_t_new` (
  `id` int NOT NULL AUTO_INCREMENT,
  `v` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```
第三步、在新表上加列。
```sql
ALTER TABLE `tempdb`.`_t_new` add column i int not null default 0;
```
第四步、创建触发器把源表上的变更都同步到新表。
```sql

CREATE TRIGGER `pt_osc_tempdb_t_del` AFTER DELETE ON `tempdb`.`t` FOR EACH ROW DELETE IGNORE FROM `tempdb`.`_t_new` WHERE `tempdb`.`_t_new`.`id` <=> OLD.`id`;

CREATE TRIGGER `pt_osc_tempdb_t_upd` AFTER UPDATE ON `tempdb`.`t` FOR EACH ROW BEGIN DELETE IGNORE FROM `tempdb`.`_t_new` WHERE !(OLD.`id` <=> NEW.`id`) AND `tempdb`.`_t_new`.`id` <=> OLD.`id`;REPLACE INTO `tempdb`.`_t_new` (`id`, `v`) VALUES (NEW.`id`, NEW.`v`);

CREATE TRIGGER `pt_osc_tempdb_t_ins` AFTER INSERT ON `tempdb`.`t` FOR EACH ROW REPLACE INTO `tempdb`.`_t_new` (`id`, `v`) VALUES (NEW.`id`, NEW.`v`);

```
第五步、同步数据并更新表的统计信息。
```sql

INSERT LOW_PRIORITY IGNORE INTO `tempdb`.`_t_new` (`id`, `v`) SELECT `id`, `v` FROM `tempdb`.`t` LOCK IN SHARE MODE /*pt-online-schema-change 35883 copy table*/

ANALYZE TABLE `tempdb`.`_t_new`;
```
第六步、重命名表并删除老表和触发器。
```sql
RENAME TABLE `tempdb`.`t` TO `tempdb`.`_t_old`, `tempdb`.`_t_new` TO `tempdb`.`t`;
DROP TABLE IF EXISTS `tempdb`.`_t_old`;
DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_del`;
DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_upd`;
DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_ins`;
```

google-adsense

---

## 完整的 pt-online-schema-change 变更日志
```log
2020-06-24T15:14:38.575675+08:00          207 Connect   superdev@127.0.0.1 on tempdb using TCP/IP
2020-06-24T15:14:38.576605+08:00          207 Query     SHOW VARIABLES LIKE 'innodb\_lock_wait_timeout'
2020-06-24T15:14:38.578790+08:00          207 Query     SET SESSION innodb_lock_wait_timeout=1
2020-06-24T15:14:38.579306+08:00          207 Query     SHOW VARIABLES LIKE 'lock\_wait_timeout'
2020-06-24T15:14:38.580888+08:00          207 Query     SET SESSION lock_wait_timeout=60
2020-06-24T15:14:38.581351+08:00          207 Query     SHOW VARIABLES LIKE 'wait\_timeout'
2020-06-24T15:14:38.582958+08:00          207 Query     SET SESSION wait_timeout=10000
2020-06-24T15:14:38.583417+08:00          207 Query     SELECT @@SQL_MODE
2020-06-24T15:14:38.583835+08:00          207 Query     SET @@SQL_QUOTE_SHOW_CREATE = 1/*!40101, @@SQL_MODE='NO_AUTO_VALUE_ON_ZERO,ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'*/
2020-06-24T15:14:38.584410+08:00          207 Query     SELECT VERSION()
2020-06-24T15:14:38.584861+08:00          207 Query     SHOW VARIABLES LIKE 'character_set_server'
2020-06-24T15:14:38.586737+08:00          207 Query     SET NAMES 'utf8mb4'
2020-06-24T15:14:38.587323+08:00          207 Query     SELECT @@server_id /*!50038 , @@hostname*/
2020-06-24T15:14:38.588756+08:00          208 Connect   superdev@127.0.0.1 on tempdb using TCP/IP
2020-06-24T15:14:38.589446+08:00          208 Query     SHOW VARIABLES LIKE 'innodb\_lock_wait_timeout'
2020-06-24T15:14:38.591982+08:00          208 Query     SET SESSION innodb_lock_wait_timeout=1
2020-06-24T15:14:38.592563+08:00          208 Query     SHOW VARIABLES LIKE 'lock\_wait_timeout'
2020-06-24T15:14:38.594339+08:00          208 Query     SET SESSION lock_wait_timeout=60
2020-06-24T15:14:38.594915+08:00          208 Query     SHOW VARIABLES LIKE 'wait\_timeout'
2020-06-24T15:14:38.596842+08:00          208 Query     SET SESSION wait_timeout=10000
2020-06-24T15:14:38.597735+08:00          208 Query     SELECT @@SQL_MODE
2020-06-24T15:14:38.598303+08:00          208 Query     SET @@SQL_QUOTE_SHOW_CREATE = 1/*!40101, @@SQL_MODE='NO_AUTO_VALUE_ON_ZERO,ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'*/
2020-06-24T15:14:38.598807+08:00          208 Query     SELECT VERSION()
2020-06-24T15:14:38.599332+08:00          208 Query     SHOW VARIABLES LIKE 'character_set_server'
2020-06-24T15:14:38.600979+08:00          208 Query     SET NAMES 'utf8mb4'
2020-06-24T15:14:38.601536+08:00          208 Query     SELECT @@server_id /*!50038 , @@hostname*/
2020-06-24T15:14:38.601920+08:00          207 Query     SHOW VARIABLES LIKE 'wsrep_on'
2020-06-24T15:14:38.603851+08:00          207 Query     SHOW VARIABLES LIKE 'version%'
2020-06-24T15:14:38.605759+08:00          207 Query     SHOW ENGINES
2020-06-24T15:14:38.606751+08:00          207 Query     SHOW VARIABLES LIKE 'innodb_version'
2020-06-24T15:14:38.608990+08:00          207 Query     SHOW VARIABLES LIKE 'innodb_stats_persistent'
2020-06-24T15:14:38.610874+08:00          207 Query     SELECT @@SERVER_ID
2020-06-24T15:14:38.611662+08:00          207 Query     SHOW GRANTS FOR CURRENT_USER()
2020-06-24T15:14:38.612703+08:00          207 Query     SHOW FULL PROCESSLIST
2020-06-24T15:14:38.613634+08:00          207 Query     SHOW SLAVE HOSTS
2020-06-24T15:14:38.615002+08:00          207 Query     SHOW GLOBAL STATUS LIKE 'Threads_running'
2020-06-24T15:14:38.616995+08:00          207 Query     SHOW GLOBAL STATUS LIKE 'Threads_running'
2020-06-24T15:14:38.620423+08:00          207 Query     SELECT CONCAT(@@hostname, @@port)
2020-06-24T15:14:38.621603+08:00          207 Query     SHOW TABLES FROM `tempdb` LIKE 't'
2020-06-24T15:14:38.624640+08:00          207 Query     SELECT VERSION()
2020-06-24T15:14:38.625454+08:00          207 Query     SHOW TRIGGERS FROM `tempdb` LIKE 't'
2020-06-24T15:14:38.627412+08:00          207 Query     /*!40101 SET @OLD_SQL_MODE := @@SQL_MODE, @@SQL_MODE := '', @OLD_QUOTE := @@SQL_QUOTE_SHOW_CREATE, @@SQL_QUOTE_SHOW_CREATE := 1 */
2020-06-24T15:14:38.628107+08:00          207 Query     USE `tempdb`
2020-06-24T15:14:38.628692+08:00          207 Query     SHOW CREATE TABLE `tempdb`.`t`
2020-06-24T15:14:38.629342+08:00          207 Query     /*!40101 SET @@SQL_MODE := @OLD_SQL_MODE, @@SQL_QUOTE_SHOW_CREATE := @OLD_QUOTE */
2020-06-24T15:14:38.630045+08:00          207 Query     EXPLAIN SELECT * FROM `tempdb`.`t` WHERE 1=1
2020-06-24T15:14:38.631817+08:00          207 Query     SELECT table_schema, table_name FROM information_schema.key_column_usage WHERE referenced_table_schema='tempdb' AND referenced_table_name='t'
2020-06-24T15:14:38.646152+08:00          207 Query     SHOW VARIABLES LIKE 'version%'
2020-06-24T15:14:38.649375+08:00          207 Query     SHOW ENGINES
2020-06-24T15:14:38.650198+08:00          207 Query     SHOW VARIABLES LIKE 'innodb_version'
2020-06-24T15:14:38.653119+08:00          207 Query     SELECT table_schema, table_name FROM information_schema.key_column_usage WHERE referenced_table_schema='tempdb' AND referenced_table_name='t'
2020-06-24T15:14:38.667653+08:00          207 Query     SHOW VARIABLES LIKE 'wsrep_on'
2020-06-24T15:14:38.669612+08:00          207 Query     /*!40101 SET @OLD_SQL_MODE := @@SQL_MODE, @@SQL_MODE := '', @OLD_QUOTE := @@SQL_QUOTE_SHOW_CREATE, @@SQL_QUOTE_SHOW_CREATE := 1 */
2020-06-24T15:14:38.670054+08:00          207 Query     USE `tempdb`
2020-06-24T15:14:38.670730+08:00          207 Query     SHOW CREATE TABLE `tempdb`.`t`
2020-06-24T15:14:38.674135+08:00          207 Query     /*!40101 SET @@SQL_MODE := @OLD_SQL_MODE, @@SQL_QUOTE_SHOW_CREATE := @OLD_QUOTE */
2020-06-24T15:14:38.677819+08:00          207 Query     CREATE TABLE `tempdb`.`_t_new` (
  `id` int NOT NULL AUTO_INCREMENT,
  `v` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
2020-06-24T15:14:38.690825+08:00          207 Query     ALTER TABLE `tempdb`.`_t_new` add column i int not null default 0
2020-06-24T15:14:38.699578+08:00          207 Query     /*!40101 SET @OLD_SQL_MODE := @@SQL_MODE, @@SQL_MODE := '', @OLD_QUOTE := @@SQL_QUOTE_SHOW_CREATE, @@SQL_QUOTE_SHOW_CREATE := 1 */
2020-06-24T15:14:38.700142+08:00          207 Query     USE `tempdb`
2020-06-24T15:14:38.700688+08:00          207 Query     SHOW CREATE TABLE `tempdb`.`_t_new`
2020-06-24T15:14:38.701818+08:00          207 Query     /*!40101 SET @@SQL_MODE := @OLD_SQL_MODE, @@SQL_QUOTE_SHOW_CREATE := @OLD_QUOTE */
2020-06-24T15:14:38.703640+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'DELETE'    AND ACTION_TIMING = 'AFTER'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.705646+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'UPDATE'    AND ACTION_TIMING = 'AFTER'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.707832+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'INSERT'    AND ACTION_TIMING = 'AFTER'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.710663+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'DELETE'    AND ACTION_TIMING = 'BEFORE'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.712522+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'UPDATE'    AND ACTION_TIMING = 'BEFORE'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.714101+08:00          207 Query     SELECT TRIGGER_SCHEMA, TRIGGER_NAME, DEFINER, ACTION_STATEMENT, SQL_MODE,        CHARACTER_SET_CLIENT, COLLATION_CONNECTION, EVENT_MANIPULATION, ACTION_TIMING   FROM INFORMATION_SCHEMA.TRIGGERS  WHERE EVENT_MANIPULATION = 'INSERT'    AND ACTION_TIMING = 'BEFORE'    AND TRIGGER_SCHEMA = 'tempdb'    AND EVENT_OBJECT_TABLE = 't'
2020-06-24T15:14:38.715866+08:00          207 Query     CREATE TRIGGER `pt_osc_tempdb_t_del` AFTER DELETE ON `tempdb`.`t` FOR EACH ROW DELETE IGNORE FROM `tempdb`.`_t_new` WHERE `tempdb`.`_t_new`.`id` <=> OLD.`id`
2020-06-24T15:14:38.727056+08:00          207 Query     CREATE TRIGGER `pt_osc_tempdb_t_upd` AFTER UPDATE ON `tempdb`.`t` FOR EACH ROW BEGIN DELETE IGNORE FROM `tempdb`.`_t_new` WHERE !(OLD.`id` <=> NEW.`id`) AND `tempdb`.`_t_new`.`id` <=> OLD.`id`;REPLACE INTO `tempdb`.`_t_new` (`id`, `v`) VALUES (NEW.`id`, NEW.`v`);END
2020-06-24T15:14:38.734312+08:00          207 Query     CREATE TRIGGER `pt_osc_tempdb_t_ins` AFTER INSERT ON `tempdb`.`t` FOR EACH ROW REPLACE INTO `tempdb`.`_t_new` (`id`, `v`) VALUES (NEW.`id`, NEW.`v`)
2020-06-24T15:14:38.742395+08:00          207 Query     EXPLAIN SELECT * FROM `tempdb`.`t` WHERE 1=1
2020-06-24T15:14:38.745021+08:00          207 Query     EXPLAIN SELECT `id`, `v` FROM `tempdb`.`t` LOCK IN SHARE MODE /*explain pt-online-schema-change 35883 copy table*/
2020-06-24T15:14:38.746343+08:00          207 Query     INSERT LOW_PRIORITY IGNORE INTO `tempdb`.`_t_new` (`id`, `v`) SELECT `id`, `v` FROM `tempdb`.`t` LOCK IN SHARE MODE /*pt-online-schema-change 35883 copy table*/
2020-06-24T15:14:38.749994+08:00          207 Query     SHOW WARNINGS
2020-06-24T15:14:38.751572+08:00          207 Query     SHOW GLOBAL STATUS LIKE 'Threads_running'
2020-06-24T15:14:38.760435+08:00          207 Query     ANALYZE TABLE `tempdb`.`_t_new` /* pt-online-schema-change */
2020-06-24T15:14:38.766956+08:00          207 Query     RENAME TABLE `tempdb`.`t` TO `tempdb`.`_t_old`, `tempdb`.`_t_new` TO `tempdb`.`t`
2020-06-24T15:14:38.794146+08:00          207 Query     DROP TABLE IF EXISTS `tempdb`.`_t_old`
2020-06-24T15:14:38.800747+08:00          207 Query     DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_del`
2020-06-24T15:14:38.802445+08:00          207 Query     DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_upd`
2020-06-24T15:14:38.807020+08:00          207 Query     DROP TRIGGER IF EXISTS `tempdb`.`pt_osc_tempdb_t_ins`
2020-06-24T15:14:38.812046+08:00          207 Query     SHOW TABLES FROM `tempdb` LIKE '\_t\_new'
2020-06-24T15:14:38.821084+08:00          208 Quit      
2020-06-24T15:14:38.823823+08:00          207 Quit
```

---

