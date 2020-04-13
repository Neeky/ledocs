## MySQL-8.0.19 新特性防止暴力破解
MySQL-8.0.19 在账号安全上做了增强，现在可以支持当用户登录失败多少次(因密码错误而失败)后就直接锁定这个用户，不让它登录了。

![user-management](static/2020-16/user-management.png)

google-adsense

---

## 体验新特性
MySQL-8.0.19 在 `create user` 和 `alter user` 上加了两个子句，分别用于指定错误次数，和锁定时间。

```sql
mysql> create user appuser@'127.0.0.1' identified by '123456' failed_login_attempts 3 password_lock_time 30; 
Query OK, 0 rows affected (0.01 sec)
```
failed_login_attempts 用于指定登录失败的次数。

password_lock_time 用于设定当失败次数达到指定次数之后要锁定多久(单位天)。

---

验证效果。

```bash
mysql -h127.0.0.1 -P3306 -uappuser -p123                                      
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): Access denied for user 'appuser'@'127.0.0.1' (using password: YES)

mysql -h127.0.0.1 -P3306 -uappuser -p123
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): Access denied for user 'appuser'@'127.0.0.1' (using password: YES)

mysql -h127.0.0.1 -P3306 -uappuser -p123
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 3955 (HY000): Access denied for user 'appuser'@'127.0.0.1'. Account is blocked for 30 day(s) (30 day(s) remaining) due to 3 consecutive failed logins.
```
可以看到当用户失败 3 次之后被锁定了 30 天。

---

## 官方文档
官方更新日志 [Account Management Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-19.html#mysqld-8-0-19-account-management) 。