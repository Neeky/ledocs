## 概要
MySQL-8.0.19 发布的第一时间 dbm 就提供了支持，那 dbm 自动化安装的 MySQL 实例在性能上是一个怎样的表现呢？我的测试环境如下。

```
|**IP**|**CPU**|**Mem**|**Disk**|**系统版本**|**MySQL版本**|**角色**|
|------|-------|-------|--------|-----------|------------|--------|
|192.168.100.10|32(逻辑核心)|128G|4TSSD   |centos-7.6 | MySQL-8.0.19| Master|
|192.168.100.20|32(逻辑核心)|128G|4TSSD   |centos-7.6 | MySQL-8.0.19| Slave |

```

---

dbm-agent 在安装 MySQL 时会根据主机配置自动的完成参数优化，默认情况下的性能表现如下图。

![](static/2020-14/mysql-8.0.19-015.png)


google-adsense

---

## 安装
安装 Master。
```bash
dbma-cli-single-instance --port=3306 --max-mem=131072 install
```
安装 Slave。
```bash
dbma-cli-build-slave --host=192.168.100.10 --port=3306 --max-mem=131072 build-slave
```
验证同步是否正常。
```bash
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.100.10
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000051
          Read_Master_Log_Pos: 88792484
               Relay_Log_File: relay.000078
                Relay_Log_Pos: 433520819
        Relay_Master_Log_File: mysql-bin.000027
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```
dbm-agent 自动生成的配置文件如下。
```ini
[mysqld]
#### for basic
user                                     = mysql3306
basedir                                  = /usr/local/mysql-8.0.19-linux-glibc2.12-x86_64
datadir                                  = /database/mysql/data/3306
server_id                                = 1794
port                                     = 3306
bind_address                             = *
admin_address                            = 127.0.0.1
mysqlx_port                              = 33060
admin_port                               = 33062
socket                                   = /tmp/mysql-3306.sock
mysqlx_socket                            = /tmp/mysqlx-33060.sock
pid_file                                 = /tmp/mysql-3306.pid  
character_set_server                     = utf8mb4
open_files_limit                         = 102000
max_prepared_stmt_count                  = 1048576  
skip_name_resolve                        = 1     
super_read_only                          = OFF
log_timestamps                           = system
event_scheduler                          = OFF
auto_generate_certs                      = ON
activate_all_roles_on_login              = ON
end_markers_in_json                      = OFF
tmpdir                                   = /tmp/
max_connections                          = 1024
autocommit                               = ON
sort_buffer_size                         = 256K
join_buffer_size                         = 256K
eq_range_index_dive_limit                = 200

#### for table 
big_tables                               = OFF
sql_require_primary_key                  = OFF
lower_case_table_names                   = 1
auto_increment_increment                 = 1
auto_increment_offset                    = 1
table_open_cache                         = 4000
table_definition_cache                   = 2000
table_open_cache_instances               = 32

#### for net 
max_allowed_packet                       = 1G
connect_timeout                          = 10
interactive_timeout                      = 28800
net_read_timeout                         = 30
net_retry_count                          = 10
net_write_timeout                        = 60
net_buffer_length                        = 32K

#### for logs 
log_output                               = FILE
general_log                              = OFF
general_log_file                         = general.log
## error
log_error                                = err.log
log_statements_unsafe_for_binlog         = ON
## slow
slow_query_log                           = ON
slow_query_log_file                      = slow.log
long_query_time                          = 2
log_queries_not_using_indexes            = OFF
log_slow_admin_statements                = OFF
log_slow_slave_statements                = OFF
## binlog
log_bin                                  = /binlog/mysql/binlog/3306/mysql-bin
binlog_checksum                          = none
log_bin_trust_function_creators          = ON
binlog_direct_non_transactional_updates  = OFF
binlog_expire_logs_seconds               = 604800
binlog_error_action                      = ABORT_SERVER
binlog_format                            = ROW
max_binlog_stmt_cache_size               = 1G
max_binlog_cache_size                    = 1G
max_binlog_size                          = 1G
binlog_order_commits                     = ON
binlog_row_image                         = FULL
binlog_row_metadata                      = MINIMAL
binlog_rows_query_log_events             = ON
binlog_stmt_cache_size                   = 32K
log_slave_updates                        = ON
binlog_transaction_dependency_history_size =25000
binlog_transaction_dependency_tracking   = WRITESET
sync_binlog                              = 1
binlog_cache_size                        = 96K
binlog_group_commit_sync_delay           = 0
binlog_group_commit_sync_no_delay_count  = 0

#### for replication
rpl_semi_sync_master_enabled             = 1
rpl_semi_sync_slave_enabled              = 1
rpl_semi_sync_master_timeout             = 1000
rpl_semi_sync_master_wait_point          = AFTER_SYNC
rpl_semi_sync_master_wait_no_slave       = ON
rpl_semi_sync_master_wait_for_slave_count = 1
master_info_repository                   = table
sync_master_info                         = 10000
skip_slave_start                         = OFF
slave_load_tmpdir                        = /tmp/
plugin_load_add                          = semisync_master.so
plugin_load_add                          = semisync_slave.so
relay_log                                = relay
sync_relay_log                           = 10000
sync_relay_log_info                      = 10000
relay_log_info_repository                = table
slave_preserve_commit_order              = ON
slave_parallel_type                      = logical_clock
slave_parallel_workers                   = 2
slave_max_allowed_packet                 = 1G



#### for gtid
gtid_mode                                = ON
binlog_gtid_simple_recovery              = ON
enforce_gtid_consistency                 = ON
gtid_executed_compression_period         = 1000

#### for clone 
plugin-load-add                          = mysql_clone.so
clone                                    = FORCE_PLUS_PERMANEN

#### for engines 
default_storage_engine                   = innodb
default_tmp_storage_engine               = innodb
internal_tmp_mem_storage_engine          = TempTable

#### for innodb
innodb_data_home_dir                     = ./
innodb_data_file_path                    = ibdata1:64M:autoextend
innodb_page_size                         = 16K
innodb_default_row_format                = dynamic
innodb_log_group_home_dir                = ./
innodb_redo_log_encrypt                  = OFF
innodb_online_alter_log_max_size         = 256M
innodb_undo_directory                    = ./
innodb_undo_log_encrypt                  = OFF
innodb_undo_log_truncate                 = ON
innodb_max_undo_log_size                 = 1G
innodb_rollback_on_timeout               = OFF
innodb_rollback_segments                 = 128
innodb_log_checksums                     = ON
innodb_checksum_algorithm                = crc32
innodb_log_compressed_pages              = ON
innodb_doublewrite                       = ON
innodb_commit_concurrency                = 0
innodb_read_only                         = OFF
innodb_dedicated_server                  = OFF
innodb_old_blocks_pct                    = 37
innodb_old_blocks_time                   = 1000
innodb_random_read_ahead                 = OFF
innodb_read_ahead_threshold              = 56
innodb_max_dirty_pages_pct_lwm           = 20
innodb_max_dirty_pages_pct               = 90
innodb_lru_scan_depth                    = 1024
innodb_adaptive_flushing                 = ON
innodb_adaptive_flushing_lwm             = 10
innodb_flushing_avg_loops                = 30
innodb_buffer_pool_dump_pct              = 50
innodb_buffer_pool_dump_at_shutdown      = ON
innodb_buffer_pool_load_at_startup       = ON
innodb_buffer_pool_filename              = ib_buffer_pool
innodb_stats_persistent                  = ON
innodb_stats_on_metadata                 = ON
innodb_stats_method                      = nulls_equal
innodb_stats_auto_recalc                 = ON
innodb_stats_include_delete_marked       = ON
innodb_stats_persistent_sample_pages     = 20
innodb_stats_transient_sample_pages      = 8
innodb_status_output                     = OFF
innodb_status_output_locks               = OFF
innodb_buffer_pool_dump_now              = OFF
innodb_buffer_pool_load_abort            = OFF
innodb_buffer_pool_load_now              = OFF
innodb_thread_concurrency                = 0
innodb_concurrency_tickets               = 5000
innodb_thread_sleep_delay                = 15000
innodb_adaptive_max_sleep_delay          = 150000
innodb_read_io_threads                   = 8
innodb_write_io_threads                  = 4
innodb_use_native_aio                    = ON
innodb_flush_sync                        = OFF
innodb_spin_wait_delay                   = 6
innodb_purge_threads                     = 6
innodb_purge_batch_size                  = 300
innodb_purge_rseg_truncate_frequency     = 128
innodb_deadlock_detect                   = ON
innodb_print_all_deadlocks               = ON
innodb_lock_wait_timeout                 = 50
innodb_table_locks                       = ON
innodb_sync_array_size                   = 1
innodb_sync_spin_loops                   = 30
innodb_print_ddl_logs                    = OFF
innodb_replication_delay                 = 0
innodb_cmp_per_index_enabled             = OFF
innodb_disable_sort_file_cache           = OFF
innodb_numa_interleave                   = OFF
innodb_strict_mode                       = ON
innodb_sort_buffer_size                  = 1M
innodb_fast_shutdown                     = 1
innodb_force_load_corrupted              = OFF
innodb_force_recovery                    = 0
innodb_temp_tablespaces_dir              = ./#innodb_temp/
innodb_tmpdir                            = ./
innodb_temp_data_file_path               = ibtmp1:64M:autoextend
innodb_page_cleaners                     = 4
innodb_adaptive_hash_index               = ON
innodb_adaptive_hash_index_parts         = 8
innodb_flush_log_at_timeout              = 1
innodb_fsync_threshold                   = 0
innodb_fill_factor                       = 90
innodb_file_per_table                    = ON
innodb_autoextend_increment              = 64
innodb_open_files                        = 100000
innodb_buffer_pool_chunk_size            = 128M
innodb_buffer_pool_instances             = 32
innodb_log_files_in_group                = 8
innodb_log_file_size                     = 256M
innodb_flush_neighbors                   = 0
innodb_io_capacity                       = 4000
innodb_io_capacity_max                   = 20000
innodb_autoinc_lock_mode                 = 2
innodb_change_buffer_max_size            = 25
innodb_flush_method                      = O_DIRECT
innodb_log_buffer_size                   = 2G
innodb_flush_log_at_trx_commit           = 1
innodb_buffer_pool_size                  = 96G





####  for performance_schema
performance_schema                                                      =ON  
performance_schema_consumer_global_instrumentation                      =ON  
performance_schema_consumer_thread_instrumentation                      =ON  
performance_schema_consumer_events_stages_current                       =ON  
performance_schema_consumer_events_stages_history                       =ON  
performance_schema_consumer_events_stages_history_long                  =OFF 
performance_schema_consumer_statements_digest                           =ON  
performance_schema_consumer_events_statements_current                   =ON  
performance_schema_consumer_events_statements_history                   =ON  
performance_schema_consumer_events_statements_history_long              =OFF 
performance_schema_consumer_events_waits_current                        =ON  
performance_schema_consumer_events_waits_history                        =ON  
performance_schema_consumer_events_waits_history_long                   =OFF 
performance-schema-instrument                                           ='memory/%=COUNTED'








# -- ~ _ ~    ~ _ ~     ~ _ ~ -- 
# base on mysql-8.0.19
# generated by https://www.sqlpy.com 2020年1月15日 10:32
# wechat: jianglegege
# email: 1721900707@qq.com
# -- ~ _ ~ --   


```

---

## 压测
用 sysbench 对 MySQL 进行压力测试。

准备数据。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=120 \
oltp_read_write prepare
```

性能压测。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=120 \
oltp_read_write run
```

---
 
MySQL 的性能表现。

![](static/2020-14/mysql-8.0.19-001.jpeg)

操作系统层面的资源消耗情况。

![](static/2020-14/mysql-8.0.19-002.png)

从资源使用情况来看 120 个并发并没有把主机的性能完全利用起来，接下来准备一步步加大并发。

---

## 150并发
把并发加大到 150。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=150 \
oltp_read_write run
```

提高并发后 select 由之前的 46k 提升到 57k，其它类型的处理能力也得到了提升。 

![](static/2020-14/mysql-8.0.19-003.png)

资源使用率上升。

![](static/2020-14/mysql-8.0.19-004.png)

总体上看负载还行， iowait 也比较小，可以继续加大并发。

---

## 180并发
把并发加大到 180。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=180 \
oltp_read_write run
```
提高并发后 select 由之前的 57k 提升到 67k，其它类型的处理能力也得到了提升。 

![](static/2020-14/mysql-8.0.19-005.png)

资源使用率上升到 70%。

![](static/2020-14/mysql-8.0.19-006.png)

---


## 210并发
把并发加大到 210。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=210 \
oltp_read_write run
```
提高并发后 select 由之前的 67k 提升到 74k，其它类型的处理能力也得到了提升。

![](static/2020-14/mysql-8.0.19-007.png)

资源使用率上升到 80%。

![](static/2020-14/mysql-8.0.19-008.png)

---

## 240并发
把并发加大到 240。

```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=240 \
oltp_read_write run
```

提高并发后 select 由之前的 74k 提升到 79k，其它类型的处理能力也得到了提升。

![](static/2020-14/mysql-8.0.19-009.png)

资源使用率上升到 85%。

![](static/2020-14/mysql-8.0.19-010.png)

---

## 270并发
把并发加大到 270。

```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=270 \
oltp_read_write run
```
当并发达到 270 的时候性能已经开始下降。

![](static/2020-14/mysql-8.0.19-011.png)

资源使用率还是 85%。

![](static/2020-14/mysql-8.0.19-012.png)

---

## 300并发
把并发加大到 300。
```bash
sysbench --mysql-host=192.168.100.10  --mysql-port=3306 \
--mysql-user=sysbench --mysql-password=sysbench --tables=120 --table_size=1000000 \
--mysql-db=tempdb --time=3600 --threads=300 \
oltp_read_write run
```

当并发达到 300 的时候性能还在下降。

![](static/2020-14/mysql-8.0.19-013.png)

同时资源使用率也稳定在 85% 左右不再下降。
![](static/2020-14/mysql-8.0.19-014.png)

---

## 结论

dbm-agent 自动生成的 my.cnf 文件还是比较机智的，不怎么用 DBA 操心，整体上并发与性能的关系如下。

![](static/2020-14/mysql-8.0.19-015.png)


---