## 问题
用 percona-toolkit 连接 MySQL-8.0.20 报如下的错误。
```bash
pt-archiver --source h=127.0.0.1,P=3306,u=root,p=xxxx,D=tempdb,t=t --dest h=127.0.0.1,t=t_backup --where "id <= 2"
DBI connect('tempdb;host=127.0.0.1;port=3306;mysql_read_default_group=client','root',...) failed: Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so: 无法打开共享对象文件: 没有那个文件或目录 at /usr/bin/pt-archiver line 2525.
```

![sqlpy](static/2020-26/sqlpy-percona-user.jpg)

---

## 分析
从报错信息可以看出是由于 pt-archiver 不支持 `caching_sha2_password` 这个密码插件引起的；如果我们用老的插件 `mysql_native_password` 的话它是支持的，那解决办法就是建一个新的用户它的密码插件设置成 `mysql_native_password` 便可。
```sql
mysql> create user nee@'127.0.0.1' identified with 'mysql_native_password' by 'dbma@0352';       
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on *.* to nee@'127.0.0.1';
Query OK, 0 rows affected (0.01 sec)
```

---

## 使用新用户
新用户就可以正常执行命令了。
```bash
pt-archiver --source h=127.0.0.1,P=3306,u=nee,p=dbma@0352,D=tempdb,t=t --dest h=127.0.0.1,t=t_backup --where "id <= 2"

```
---