## mydumper
[前面的文章](/blogs/932052756)介绍了如何源码安装 `mydumper` 下面我将通过开启 general-log 的方法来观察 mydumper 的整个备份过程中做了一些什么操作。

![sqlpy](static/2020-25/sqlpy-mydumper-02.jpg)

---

## 准备工作
```sql
create user mydumper@'127.0.0.1' identified by 'dbma@0352';
grant all on *.* to mydumper@'127.0.0.1';

create database tempdb;
use tempdb;

create table scores(  
  id int not null auto_increment primary key,
  student_name varchar(32) not null COMMENT '学生名',
  math decimal(4,1) not null COMMENT '数学',
  physis decimal(4,1) not null COMMENT '物理',
  total_score decimal(6,1) default null
);

insert into `scores` values (1,"tom",120.0,100.0,220.0);

set @@global.general_log=ON;

```
这样备份用户、库表、general_log 日志都准备好了。

---

## 最简单的备份场景
由于要通过分析 general-log 来研究 mydumper 在备份的时候都做了什么，所以我们在这里只备份 tempdb.scores 表。 mydumper 的备份命令如下。
```bash
mydumper --host=127.0.0.1 --port=3306 --user=mydumper --password=dbma@0352 --database=tempdb --outputdir /tmp/3306/ 
```
整个过程的 general-log 日志如下。
```bash
2020-05-15T01:27:01.676008+08:00           22 Connect   mydumper@127.0.0.1 on tempdb using SSL/TLS
2020-05-15T01:27:01.676409+08:00           22 Query     SET SESSION wait_timeout = 2147483
2020-05-15T01:27:01.676726+08:00           22 Query     SET SESSION net_write_timeout = 2147483
2020-05-15T01:27:01.677847+08:00           22 Query     SHOW PROCESSLIST
2020-05-15T01:27:01.678519+08:00           22 Query     FLUSH TABLES WITH READ LOCK
2020-05-15T01:27:01.682143+08:00           22 Query     START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */
2020-05-15T01:27:01.682565+08:00           22 Query     /*!40101 SET NAMES binary*/
2020-05-15T01:27:01.682936+08:00           22 Query     SHOW MASTER STATUS
2020-05-15T01:27:01.683409+08:00           22 Query     SHOW SLAVE STATUS
2020-05-15T01:27:01.687869+08:00           23 Connect   mydumper@127.0.0.1 on  using SSL/TLS
2020-05-15T01:27:01.688408+08:00           23 Query     SET SESSION wait_timeout = 2147483
2020-05-15T01:27:01.688779+08:00           23 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2020-05-15T01:27:01.689118+08:00           23 Query     START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */
2020-05-15T01:27:01.689532+08:00           23 Query     /*!40103 SET TIME_ZONE='+00:00' */
2020-05-15T01:27:01.689935+08:00           23 Query     /*!40101 SET NAMES binary*/
2020-05-15T01:27:01.694323+08:00           24 Connect   mydumper@127.0.0.1 on  using SSL/TLS
2020-05-15T01:27:01.694907+08:00           24 Query     SET SESSION wait_timeout = 2147483
2020-05-15T01:27:01.695314+08:00           24 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2020-05-15T01:27:01.695833+08:00           24 Query     START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */
2020-05-15T01:27:01.696233+08:00           24 Query     /*!40103 SET TIME_ZONE='+00:00' */
2020-05-15T01:27:01.696842+08:00           24 Query     /*!40101 SET NAMES binary*/
2020-05-15T01:27:01.702360+08:00           25 Connect   mydumper@127.0.0.1 on  using SSL/TLS
2020-05-15T01:27:01.703124+08:00           25 Query     SET SESSION wait_timeout = 2147483
2020-05-15T01:27:01.703814+08:00           25 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2020-05-15T01:27:01.704491+08:00           25 Query     START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */
2020-05-15T01:27:01.705058+08:00           25 Query     /*!40103 SET TIME_ZONE='+00:00' */
2020-05-15T01:27:01.705915+08:00           25 Query     /*!40101 SET NAMES binary*/
2020-05-15T01:27:01.712444+08:00           26 Connect   mydumper@127.0.0.1 on  using SSL/TLS
2020-05-15T01:27:01.712900+08:00           26 Query     SET SESSION wait_timeout = 2147483
2020-05-15T01:27:01.713328+08:00           26 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
2020-05-15T01:27:01.713778+08:00           26 Query     START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */
2020-05-15T01:27:01.714178+08:00           26 Query     /*!40103 SET TIME_ZONE='+00:00' */
2020-05-15T01:27:01.714567+08:00           26 Query     /*!40101 SET NAMES binary*/
2020-05-15T01:27:01.715069+08:00           22 Init DB   tempdb
2020-05-15T01:27:01.715449+08:00           22 Query     SHOW TABLE STATUS
2020-05-15T01:27:01.727164+08:00           22 Query     SHOW CREATE DATABASE `tempdb`
2020-05-15T01:27:01.727871+08:00           22 Query     UNLOCK TABLES /* FTWRL */
2020-05-15T01:27:01.728372+08:00           25 Quit      
2020-05-15T01:27:01.728517+08:00           24 Query     select COLUMN_NAME from information_schema.COLUMNS where TABLE_SCHEMA='tempdb' and TABLE_NAME='scores' and extra like '%GENERATED%'
2020-05-15T01:27:01.728625+08:00           22 Quit      
2020-05-15T01:27:01.729652+08:00           23 Query     SHOW CREATE TABLE `tempdb`.`scores`
2020-05-15T01:27:01.731830+08:00           24 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `tempdb`.`scores`
2020-05-15T01:27:01.732799+08:00           23 Quit      
2020-05-15T01:27:01.733176+08:00           26 Quit      
2020-05-15T01:27:01.733937+08:00           24 Quit
```

google-adsense

---


## mydumper 备份过程
第一步：主线程连接到数据库获取全局读锁
```sql
FLUSH TABLES WITH READ LOCK; -- 这行 SQL 执行完成之后主线程所有会话就持有了所有表的读锁，使得其它会话的写入将被阻塞。
START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */; -- 创建 innodb 层面的一致性快照
```
---

第二步：工作线程连接到数据库并创建它们自己的一致性快照
```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- 一致性快照只有 RR 级别才有
START TRANSACTION /*!40108 WITH CONSISTENT SNAPSHOT */   -- 创建 innodb 层面的一致性快照
```
因为第一步就让实现了实例级别的只读，所以第二步工作线程创建的快照所能看到的数据和第一步是一样的(innodb表)。

---

第三步：读取要备份表的元信息，并且释放表锁。
```sql
USE tempdb;
SHOW TABLE STATUS;
SHOW CREATE DATABASE `tempdb`;
UNLOCK TABLES /* FTWRL */;
```
如果你的数据库有非 innodb 的表，那么 mydumper 会在备份完 none-innodb 表之后再执行 `unlock tables;`，也就是说保证备份是一致的。

---

第四步：工作线程开始备份数据。
```sql
SHOW CREATE TABLE `tempdb`.`scores`; -- 备份表结构
SELECT /*!40001 SQL_NO_CACHE */ * FROM `tempdb`.`scores`; -- 备份数据
```

---

## mydumper 的详细选项

```bash
mydumper --help                                                             
Usage:
  mydumper [OPTION?] multi-threaded MySQL dumping

Help Options:
  -?, --help                  Show help options

Application Options:
  -B, --database              Database to dump
  -T, --tables-list           Comma delimited table list to dump (does not exclude regex option)
  -O, --omit-from-file        File containing a list of database.table entries to skip, one per line (skips before applying regex option)
  -o, --outputdir             Directory to output files to
  -s, --statement-size        Attempted size of INSERT statement in bytes, default 1000000
  -r, --rows                  Try to split tables into chunks of this many rows. This option turns off --chunk-filesize
  -F, --chunk-filesize        Split tables into chunks of this output file size. This value is in MB
  -c, --compress              Compress output files
  -e, --build-empty-files     Build dump files even if no data available from table
  -x, --regex                 Regular expression for 'db.table' matching
  -i, --ignore-engines        Comma delimited list of storage engines to ignore
  -N, --insert-ignore         Dump rows with INSERT IGNORE
  -m, --no-schemas            Do not dump table schemas with the data
  -d, --no-data               Do not dump table data
  -G, --triggers              Dump triggers
  -E, --events                Dump events
  -R, --routines              Dump stored procedures and functions
  -W, --no-views              Do not dump VIEWs
  -k, --no-locks              Do not execute the temporary shared read lock.  WARNING: This will cause inconsistent backups
  --no-backup-locks           Do not use Percona backup locks
  --less-locking              Minimize locking time on InnoDB tables.
  -l, --long-query-guard      Set long query timer in seconds, default 60
  -K, --kill-long-queries     Kill long running queries (instead of aborting)
  -D, --daemon                Enable daemon mode
  -I, --snapshot-interval     Interval between each dump snapshot (in minutes), requires --daemon, default 60
  -L, --logfile               Log file name to use, by default stdout is used
  --tz-utc                    SET TIME_ZONE='+00:00' at top of dump to allow dumping of TIMESTAMP data when a server has data in different time zones or data is being moved between servers with different time zones, defaults to on use --skip-tz-utc to disable.
  --skip-tz-utc               
  --use-savepoints            Use savepoints to reduce metadata locking issues, needs SUPER privilege
  --success-on-1146           Not increment error count and Warning instead of Critical in case of table doesnt exist
  --lock-all-tables           Use LOCK TABLE for all, instead of FTWRL
  -U, --updated-since         Use Update_time to dump only tables updated in the last U days
  --trx-consistency-only      Transactional consistency only
  --complete-insert           Use complete INSERT statements that include column names
  -h, --host                  The host to connect to
  -u, --user                  Username with the necessary privileges
  -p, --password              User password
  -a, --ask-password          Prompt For User password
  -P, --port                  TCP/IP port to connect to
  -S, --socket                UNIX domain socket file to use for connection
  -t, --threads               Number of threads to use, default 4
  -C, --compress-protocol     Use compression on the MySQL connection
  -V, --version               Show the program version and exit
  -v, --verbose               Verbosity of output, 0 = silent, 1 = errors, 2 = warnings, 3 = info, default 2
  --defaults-file             Use a specific defaults file
```

---