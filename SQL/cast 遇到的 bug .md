## 背景
SQL 语言中用 cast 函数对数据进行类型转换，语法如下：`cast(value as type);` 就这么简单个事我还遇到了问题。
```sql
mysql> select cast('123' as float);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'float)' at line 1

mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.15    |
+-----------+
1 row in set (0.00 sec)
```
语法上没有任何错误呀。

![cast](static/2020-14/cast.png)

---

## 突破
再三与官方文档对比，确认我没有写错，难道我又遇到了 bug，于是我决定在最新版本的 MySQL 上试一下。
```sql
mysql> select cast('123' as float);
+----------------------+
| cast('123' as float) |
+----------------------+
|                  123 |
+----------------------+
1 row in set (0.00 sec)

mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.19    |
+-----------+
1 row in set (0.00 sec)
```
在新版本上不再有问题，看来我是遇到了 bug 了。

---


## cast 要注意的地方
1、如果想把数据转换成整数类型，不能使用 int，官方只给出了 `unsigned` 和`sigend`，可能这个是为了统一 big int 吧。
```sql
mysql> select cast('123' as unsigned);
+-------------------------+
| cast('123' as unsigned) |
+-------------------------+
|                     123 |
+-------------------------+
1 row in set (0.00 sec)
```
2、cast 支持的类型列表如下
```bash
1、BINARY[(N)]
2、CHAR[(N)] [charset_info]
3、DATE
4、DATETIME
5、DECIMAL[(M[,D])]
6、JSON
7、NCHAR[(N)]
8、SIGNED [INTEGER]
9、UNSIGNED [INTEGER]
10、TIME
```

---