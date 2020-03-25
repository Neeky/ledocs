## dbm数据库管理中心
有没有一套开源的专门用来管理 MySQL 数据库的软件呢？还真有这就是 dbm 它能完成各类 MySQL 环境的自动部署、监控、备份等工作；dbm 的整体架构如图，这里我们主要介绍一下 dbm-agent。

google-adsense

![dbm](static/2020-12/dbm.png)

---

## dbm-agent安装与配置
dbm-agent 要安装在你用于部署 MySQL 数据库的主机上，为了方便使用 dbm-agent 提供了一些方便的命令行来完成日常任务，不强制和 dbm-center一起使用，如果你需要 web 管理页面的话那你需要 dbm-center 。 下面以单独使用 dbm-agent 为例。

```bash
#安装
sudo su 
pip3 install dbm-agent

#配置初始化
dbm-agent init

2020-03-19 10:24:31,063 INFO  user 'dbma' not exists going to create it 
2020-03-19 10:24:31,064 WARNING user group dbma not exits
2020-03-19 10:24:31,064 INFO groupadd dbma
2020-03-19 10:24:31,110 INFO create dir /usr/local/dbm-agent/
2020-03-19 10:24:31,111 INFO create config file '/usr/local/dbm-agent/etc/dbma.cnf' 
2020-03-19 10:24:31,120 INFO prepare rende init-sql-file /usr/local/dbm-agent/etc/init-users.sql
2020-03-19 10:24:31,120 INFO init-sql-file render complete
2020-03-19 10:24:31,121 INFO inseption data saved to '/usr/local/dbm-agent/logs/auto-inseption.db' 
2020-03-19 10:24:31,139 DEBUG sudo context config dbm-monitor-gateway
2020-03-19 10:24:31,372 INFO monitor-gateway render complete
2020-03-19 10:24:31,373 DEBUG sudo context config dbm-backup-proxy
2020-03-19 10:24:31,600 INFO monitor-gateway render complete
2020-03-19 10:24:31,606 INFO init complete

#初始化完成之后会自动拉起 dbm-monitor-gateway(数据库监控) 服务
systemctl status dbm-monitor-gatewayd

● dbm-monitor-gatewayd.service - dbm monitor gateway
   Loaded: loaded (/usr/lib/systemd/system/dbm-monitor-gatewayd.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2020-03-19 10:24:31 CST; 4min 55s ago
 Main PID: 1568 (dbm-monitor-gat)
   CGroup: /system.slice/dbm-monitor-gatewayd.service
           └─1568 /usr/local/python-3.8.1/bin/python3.8 /usr/local/python/bin/dbm-monitor-gate...
3月 19 10:24:31 lestudio systemd[1]: Started dbm monitor gateway.

#监控服务以守护进程运行
ps -ef | grep dbm-monitor

dbma       1568      1  0 10:24 ?        00:00:01 /usr/local/python-3.8.1/bin/python3.8 /usr/local/python/bin/dbm-monitor-gateway --monitor-user=monitor --monitor-password=dbma@0352 --bind-ip=127.0.0.1 --bind-port=8080 start

```
下载 MySQL 二进制包到 dbm-agent 包目录。
```bash
#下载 MySQL 二进制包
cd /usr/local/dbm-agent/pkg/ 
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz
```

google-adsense

---


## 自动化安装MySQL
以 dbm-agent 来安装 MySQL-8.0.19 单机为例(dbm-agent 支持主从复制、MGR 这里为了方便用单机举例)。
```bash
dbma-cli-single-instance --pkg=mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz \
--port=3306  --max-mem=128 install

2020-03-19 11:35:30,087 - dbm-agent.dbma.mysqldeploy.SingleInstanceInstaller.install - im - INFO - 1118 - execute checkings for install mysql
...
...
2020-03-19 11:35:43,384 - dbm-agent.dbma.mysqldeploy.SingleInstanceInstaller.install - im - INFO - 1153 - install mysql single instance complete
```
验证是否安装成功。
```sql
mysql -uroot -pdbma@0352 -h127.0.0.1 -P3306

mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.19    |
+-----------+
1 row in set (0.00 sec)
```
---

## 自动化监控
dbm-agent 初始化之后监控服务就在后台运行了，它会定期的扫描端口看有没有新的 MySQL 实例被安装，如果发现了新实例就把它监控起来；并把监控项用 http 协议暴露出去。
```bash
#查看当前主机上有哪些实例
curl http://127.0.0.1:8080/instances/ 
[3306]

#查看给定实例的监控项
curl http://127.0.0.1:8080/instances/3306/com_select
{
    "com_select": "90"
}

#查看给定实例的所有监控项
curl http://127.0.0.1:8080/instances/3306/
{                                                                                                
    "aborted_clients": "0",                                                                      
    "aborted_connects": "10",                                                                    
    "acl_cache_items_count": "0",
    ...
    ...
    "binlog_ignore_db": "",
    "executed_gtid_set": ""
}

```
---

## 自动化备份
因为备份是要用磁盘空间的，出于种种原因吧，备份库默认是不启动的，如果你要备份的话一个 start 就可以了。
```bash
systemctl start dbm-backup-proxyd
```
检查备份有没有成功。
```bash
tree /backup/mysql/backup/3306/
/backup/mysql/backup/3306/
└── 2020-12
    ├── 2020-03-19T11:47:46.620437-full-backup.sql
    └── binlog-position.log

1 directory, 2 files
```
当然如果你是土豪`mysqlbackup`备份也是支持的。
```bash
tree /backup/mysql/backup/3306/
/backup/mysql/backup/3306/
├── 2020-11
│   ├── 2020-03-09T10:05:02.924898-full-backup.mbi
│   ├── 2020-03-09T10:05:02.924898.log
```
---

## dbm的理念
dbm 设计之初就是为了尽一切可能来减小 DBA 的工作量，从上面的例子也可以看到只要把 dbm 相关的服务启动之后，脏活、累活、重复劳动就和 DBA 没有什么关系了；希望每一个 DBA 都可以无所事事。

dbm的官方站点：https://www.sqlpy.com/

dbm的源代码：https://github.com/Neeky/dbm-agent

---

