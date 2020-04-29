## 概要
MySQL-8.0.20 版本加上了一个压缩 `binlog` 的特性，接受了太多磁盘告警洗礼的我，感觉看到了那么一点点希望。于是在测试环境上试了下，测试环境的信息如下。

|**IP**|**CPU**|**MEM**|**SSD**|**角色**|
|------|-------|-------|-------|-------|
|192.168.100.10|32(逻辑)| 128G  | 3T    |MySQL|
|192.168.100.22|80(逻辑)| 256G  | 1T    |sysbench|


---

## 测试方法

用 sysbench 分别向 MySQL-8.0.19 和 MySQL-8.0.20 写入 1kw 行数据，观察 binlog 文件的总大小。为了简化数据库的安装操作，在这里使用 [dbm-agent](#https://github.com/Neeky/dbm-agent) 来完成 MySQL 的自动化安装。

---

## 8.0.19 场景
第一步：安装 MySQL-8.0.19
```bash
dbma-cli-single-instance --port=3306 --max-mem=40960 \
--pkg=mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz install
```
---

第二步：创建压力测试用户
```sql
create user sysbench@'%' identified by 'sysbench';
create database tempdb;
grant all on tempdb.* to sysbench@'%';
```
---

第三步：导入数据
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 --mysql-user=sysbench \
--mysql-password=sysbench --tables=100 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=120 oltp_point_select prepare
```

整个过程产生了 3.8G 的 binlog。
```bash
-rw-r----- 1 mysql3306 mysql 1.1G Apr 29 16:48 mysql-bin.000001
-rw-r----- 1 mysql3306 mysql 1.1G Apr 29 16:48 mysql-bin.000002
-rw-r----- 1 mysql3306 mysql 1.1G Apr 29 16:48 mysql-bin.000003
-rw-r----- 1 mysql3306 mysql 473M Apr 29 16:48 mysql-bin.000004
-rw-r----- 1 mysql3306 mysql  172 Apr 29 16:48 mysql-bin.index
```

---



## 8.0.20 场景
第一步：安装 MySQL-8.0.20
```bash
dbma-cli-single-instance --port=3308 --max-mem=40960 \
--pkg=mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz install
```

当 dbm-agent 检测到 MySQL 的版本大于等于 8.0.20 ，它就会开启 binlog 压缩相关的配置。

```sql
mysql> show global variables like 'binlog_transaction_compression%';
+-------------------------------------------+-------+
| Variable_name                             | Value |
+-------------------------------------------+-------+
| binlog_transaction_compression            | ON    |
| binlog_transaction_compression_level_zstd | 3     | 
+-------------------------------------------+-------+
2 rows in set (0.01 sec)
```

---

第二步：创建压力测试用户
```sql
create user sysbench@'%' identified by 'sysbench';
create database tempdb;
grant all on tempdb.* to sysbench@'%';
```
---

第三步：导入数据
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3308 --mysql-user=sysbench \
--mysql-password=sysbench --tables=100 --table_size=100000 --mysql-db=tempdb \
--time=3600 --threads=120 oltp_point_select prepare
```

整个过程产生了 1.7G 的 binlog。
```bash
-rw-r----- 1 mysql3308 mysql  171 Apr 29 16:40 mysql-bin.000001
-rw-r----- 1 mysql3308 mysql 1.1G Apr 29 16:46 mysql-bin.000002
-rw-r----- 1 mysql3308 mysql 151M Apr 29 16:46 mysql-bin.000003
-rw-r----- 1 mysql3308 mysql  129 Apr 29 16:46 mysql-bin.index
```

---

## 结论
在开启 binlog 压缩，并保持默认压缩级别(binlog_transaction_compression_level_zstd=3) 的下 MySQ-8.0.20 的 binlog 日志压缩比有 2.23；也就是说可以节约一半多的空间。

|**\\**|**MySQL-8.0.19**|**MySQL-8.0.20**|**压缩比**|
|-----|----------------|-----------------|---------|
|binlog日志大小|  3.8G | 1.7G |2.23(倍)|


---



