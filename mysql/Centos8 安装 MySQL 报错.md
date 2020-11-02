## 背景
最近有人向我反馈 dbm-agent 在 Centos-8 上安装 MySQL 后 mysql 客气端命令不能用，一执行就报错。

```bash
mysql -h192.168.196.100 -P3306  
mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
```

![sqlpy](static/2020-41/centos8-mysql.jpg)

---

## 原因
mysql 命令依赖 ncurses 来管理字符界面，在缺少 ncurses 的情况下自然就起动不了。

---

## 解析办法
centos-8 的话直接安装 yum install ncurses-compat-libs 就行。
```bash
yum install ncurses-compat-libs

...
...

已安装:
  ncurses-compat-libs-6.1-7.20180224.el8.x86_64                                                                              

完毕！
```
---

## 验收
```bash
mysql -h127.0.0.1 -P3306 -uroot -pxxxxxx                                                           
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

---