## 概要

MySQL 的 utf8 字符集确实是一个坑，之前以为自己懂了，直到最近遇到一个 4 字节的汉字，这又让我重新认识了它一次。

![](static/2021-02/lucas-favre-aj9Kd4J6XXA-unsplash.jpg)

---

## 关于编码

首先计算机并不能直接保存字符串(String)，它只能保存 0 和 1 ；为了把字符串保存下来，程序会把字符串先编码成01组成的串(字节串)，这个串是计算机器可以保存的。而这个由字符到字节的函数`y=f(x)`就是我们所说的字符集。常见的字符集有 utf8,gbk,latin1 等等。

1、如果确认一个字符串通过特定编码后的字节串呢？

对于 python3 来说 str 对象的 encode 方法可以返回编码后的字节串
```python
In [1]: s = "东临碣石，以观沧海。"

In [2]: s.encode('utf8')
Out[2]: b'\xe4\xb8\x9c\xe4\xb8\xb4\xe7\xa2\xa3\xe7\x9f\xb3\xef\xbc\x8c\xe4\xbb\xa5\xe8\xa7\x82\xe6\xb2\xa7\xe6\xb5\xb7\xe3\x80\x82'
```

2、如果是一条已经存在于数据库中的数据，我们可以用 hex 函数来检查它在数据库中的二进制串。
```sql
mysql> select x,hex(x) from t limit 1;
+--------------------------------+--------------------------------------------------------------+
| x                              | hex(x)                                                       |
+--------------------------------+--------------------------------------------------------------+
| 东临碣石，以观沧海。               | E4B89CE4B8B4E7A2A3E79FB3EFBC8CE4BBA5E8A782E6B2A7E6B5B7E38082 |
+--------------------------------+--------------------------------------------------------------+
1 row in set (0.00 sec)
```

---

## 关于版本
以下内容摘自 8.0.24 版本的 [release-note Bug fix](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-24.html#mysqld-8-0-24-bug) 的第一条
```
Important Note: ... the character set as utf8 which is becoming a synonym for utf8mb4. Now in such cases, utf8mb3 is shown instead, 
and CREATE TABLE raises the warning 'collation_name' is a collation of the deprecated character set UTF8MB3. Please consider using UTF8MB4 with an appropriate collation instead.
...
```

也就是说在 8.0.24 之前 utf8 事实上是 utf8mb3 这个就会导致一部分中文不能保存。比如 `𣌀` 字的二进制串占 4 个字节，这个时候 MySQL 就会报错。

```sql
select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.19    |
+-----------+

create table t(x varchar(16) charset utf8);

-- 
insert into t(x) values('𣌀');
ERROR 1366 (HY000): Incorrect string value: '\xF0\xA3\x8C\x80' for column 'x' at row 1

```

8.0.24 版本之后在 MySQL 中使用 utf8 会被警告，提示再这几个版本 utf8 就是 utf8mb4 了，虽然它现在还是 utf8mb3 。
```sql
mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.26    |
+-----------+

mysql> create table t( x varchar(16) charset utf8);
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> show warnings;
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                     |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 3719 | 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous. |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

希望 utf8 快点转正！

---

## 怎么办

对于新系统我们的设计应该是面向未来的，所以我建议直接就写上 `default charset utf8mb4` 对于老的系统没有必要的我们可以先不动。

---

## 调整字符集要注意的事
1、调整字符集一定是针对列来的，而不是调整表的默认字符集。
```sql
mysql> create table t(x varchar(16)) default charset utf8;

mysql> insert into t(x) values("东临碣石，以观沧海。");

mysql> select x,hex(x) as hex_utf8 from t limit 1;
+--------------------------------+--------------------------------------------------------------+
| x                              | hex_utf8                                                     |
+--------------------------------+--------------------------------------------------------------+
| 东临碣石，以观沧海。              | E4B89CE4B8B4E7A2A3E79FB3EFBC8CE4BBA5E8A782E6B2A7E6B5B7E38082 |
+--------------------------------+--------------------------------------------------------------+
```
调整表的默认字符集 MySQL 并不会对字体列进行转码。
```sql
-- 调整表的字符集
mysql> alter table t default charset gbk;

-- 可以看到列的二进制串并没有变
mysql> select x,hex(x) as hex_utf8 from t limit 1;
+--------------------------------+--------------------------------------------------------------+
| x                              | hex_utf8                                                     |
+--------------------------------+--------------------------------------------------------------+
| 东临碣石，以观沧海。              | E4B89CE4B8B4E7A2A3E79FB3EFBC8CE4BBA5E8A782E6B2A7E6B5B7E38082 |
+--------------------------------+--------------------------------------------------------------+

-- 可以看到单单只是改了一下元数据
mysql> show create table t;
+-------+----------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                             |
+-------+----------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `x` varchar(16) CHARACTER SET utf8 DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gbk |
+-------+----------------------------------------------------------------------------------------------------------+
```

正确的做法是对列进行操作，并且同时改变表的字符集(这样以后加列也方便了)。
```sql
-- 把列的字符集设置成 utf8 
mysql> alter table t modify column x varchar(16) charset gbk , default charset gbk;

-- 可以看到整个二进制串变了
mysql> select x,hex(x) as hex_utf8 from t limit 1;
+--------------------------------+------------------------------------------+
| x                              | hex_utf8                                 |
+--------------------------------+------------------------------------------+
| 东临碣石，以观沧海。              | B6ABC1D9EDD9CAAFA3ACD2D4B9DBB2D7BAA3A1A3 |
+--------------------------------+------------------------------------------+
```

---



