## 概要
[之前](#/blogs/809502061) 有做过一个 binlog 压缩能节约多少空间的测试，效果上看还是比较理想的，可以节约一半以上的空间。但是这个又引出了一个新的问题，就是这个功能对性能有多大影响呢？于是我又在测试环境度了一下。

|**IP**|**CPU**|**MEM**|**SSD**|**角色**|
|------|-------|-------|-------|-------|
|192.168.100.10|32(逻辑)| 128G  | 3T    |MySQL|
|192.168.100.22|80(逻辑)| 256G  | 1T    |sysbench|

根据之前的经验这套测试环境在 120 个表 + 240 个并发的情况，可以取得一个性能上的极大值；所以在这里就直接使用这个作为测试压力。


---


## 8.0.19 场景
第一步：安装。
```bash
dbma-cli-single-instance --port=3306 --max-mem=131072 \
--pkg=mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz install
```
第二步：创建测试用户。
```sql
create user sysbench@'%' identified by 'sysbench';
create database tempdb;
grant all on tempdb.* to sysbench@'%';
```
第三步：填充数据并进行压力测试。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=120 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=240 oltp_point_select prepare

sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=120 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=240 oltp_point_select run
```

性能表现。

![binlog-001](static/2020-18/binlog-001.png)

资源消耗情况。

![binlog-002](static/2020-18/binlog-002.png)

---

## 8.0.20 + binlog 压缩
第一步：安装。
```bash
dbma-cli-single-instance --port=3306 --max-mem=131072 \
--pkg=mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz install
```
第二步：创建测试用户。
```sql
create user sysbench@'%' identified by 'sysbench';
create database tempdb;
grant all on tempdb.* to sysbench@'%';

-- dbm-agent 默认会开启 binlog 压缩
show global variables like 'binlog_transaction_compression%';
+-------------------------------------------+-------+
| Variable_name                             | Value |
+-------------------------------------------+-------+
| binlog_transaction_compression            | ON    |
| binlog_transaction_compression_level_zstd | 3     |
+-------------------------------------------+-------+
2 rows in set (0.00 sec)
```
第三步：填充数据并进行压力测试。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=120 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=240 oltp_point_select prepare

sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=120 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=240 oltp_point_select run
```

性能表现。

![binlog-003](static/2020-18/binlog-003.png)

资源消耗情况。

![binlog-004](static/2020-18/binlog-004.png)

---
## 8.0.20 + binlog 不压缩
第一步: 关闭 binlog 压缩功能。
```sql
set @@global.binlog_transaction_compression='OFF';

show global variables like 'binlog_transaction_compression%';
+-------------------------------------------+-------+
| Variable_name                             | Value |
+-------------------------------------------+-------+
| binlog_transaction_compression            | OFF   |
| binlog_transaction_compression_level_zstd | 3     |
+-------------------------------------------+-------+
2 rows in set (0.01 sec)
```

第二步：进行压力测试。

```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=120 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=240 oltp_point_select run
```

性能表现。

![binlog-005](static/2020-18/binlog-005.png)

资源消耗情况。

![binlog-006](static/2020-18/binlog-006.png)

---

## 结论
开启 binlog 压缩会对性能有影响，大概会让性能下降 `1%`，cpu 多消耗 `1%`。

|****          |**MySQL-8.0.19**|**MySQL-8.0.20 binlog压缩**|**MySQL-8.0.20 binlog不压缩**|
|----------------|----------------|--------------------|---------------------|
|**select(qps)** |81.3k           |79.6k               | 80.7k               |
|**delete(qps)** |5.7k            |5.6k                | 5.7k                |
|**insert(qps)** |5.7k            |5.6k                | 5.7k                |
|**udate(qps)**  |11.5k           |11.3k               | 11.5k               |
|**cpu(qps)**    |87%             |89%                 | 88%                 |


---
