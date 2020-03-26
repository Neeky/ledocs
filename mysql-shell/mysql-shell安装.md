## 概要
`mysql-shell` 是一个新的客户端软件，与之前的 `mysql` 命令不同的是 `mysql-shell` 支持使用 `SQL` 、`js`、 `python` 这三种语言与 `MySQL-Server`交互。

![mysql-shell](static/2020-13/mysql-shell.png)

google-adsense

---

## 安装
MySQL 官方提供了 `mysql-shell` 的绿色版本(解压就能用的版本)，下面的代码会自动安装`mysql-shell-8.0.19`。
```bash
sudo su

cd /tmp/
wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.19-linux-glibc2.12-x86-64bit.tar.gz
tar -xvf mysql-shell-8.0.19-linux-glibc2.12-x86-64bit.tar.gz -C /usr/local

cd /usr/local/
ln -s mysql-shell-8.0.19-linux-glibc2.12-x86-64bit mysql-shell

echo 'export PATH=/usr/local/mysql-shell/bin:$PATH' >> /etc/profile 
source /etc/profile
```

---

## 验证是否安装成功
如果可以正常的执行 `mysqlsh` 命令说明 `mysql-shell` 安装成功了。
```bash
mysqlsh
MySQL Shell 8.0.19

Copyright (c) 2016, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.                                                                              
```
如果想要退出 mysqlsh 的话可以直接输入 `\quit` 来完成。
```bash
mysqlsh
MySQL Shell 8.0.19

Copyright (c) 2016, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.
 MySQL  JS > \quit                                                                               
Bye!
```
---

## 查看当前会话的状态
mysqlsh 通过一个叫 `session` 的对象连接到` MySQL-Server`，session 是全局的并且是跨语言的，如果当前的 mysqlsh 没有连接到任何 `MySQL-Server` 那么可以看到如下状态。
```js
MySQL  JS > session == null;                                                                    
true
MySQL  JS > shell.status();                                                                     
MySQL Shell version 8.0.19

Not Connected.
```
---