
## 背景

就 innodb 表压缩而言包含了两大类，一类叫 [table compression](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-usage.html)另一类叫 [transparent page compression](https://dev.mysql.com/doc/refman/8.0/en/innodb-page-compression.html) 。 那他们的效果怎么样呢？

![transparent-page-compression.jpg](static/2021-01/transparent-page-compression.jpg)

---

## 结论

transparent page compression 在节约磁盘空间上并没有什么卵用，有时候还有可能会占用更加多的磁盘空间(innodb_page_size=32k时)。

|**类型**|**行数**|**表空间文件大小**|
|----|----|----|
|table_compression|200000| 22MB|
|transparent_page_compression|200000|29M|
|normal|200000|29M|


环境信息如下

```sql
mysql> select @@version;
+-----------+
| @@version |
+-----------+
| 8.0.23    |
+-----------+
1 row in set (0.00 sec)

mysql> show global variables like 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.00 sec)

mysql> show global variables like 'innodb_file_per%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)
```

---


## table-compression
```sql
CREATE TABLE employees_with_innodb_table_compression (
    emp_no      BIGINT          NOT NULL,
    birth_date  DATE            NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    last_name   VARCHAR(16)     NOT NULL,
    gender      ENUM ('M','F')  NOT NULL,
    hire_date   DATE            NOT NULL,
    PRIMARY KEY (emp_no)
) ROW_FORMAT=COMPRESSED ,KEY_BLOCK_SIZE=8;

mysql> select count(*) from employees_with_innodb_table_compression;
+----------+
| count(*) |
+----------+
|   200000 |
+----------+
1 row in set (0.01 sec)

-rw-r-----. 1 mysql3307 mysql 22M 2月  19 19:04 employees_with_innodb_table_compression.ibd

```

---

## transparent_page_compression
```sql
CREATE TABLE employees_with_transparent_page_compression (
    emp_no      BIGINT          NOT NULL,
    birth_date  DATE            NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    last_name   VARCHAR(16)     NOT NULL,
    gender      ENUM ('M','F')  NOT NULL,
    hire_date   DATE            NOT NULL,
    PRIMARY KEY (emp_no)
) COMPRESSION="zlib";

mysql> select count(*) from employees_with_transparent_page_compression;
+----------+
| count(*) |
+----------+
|   200000 |
+----------+
1 row in set (0.02 sec)

-rw-r-----. 1 mysql3307 mysql 29M 2月  19 19:23 employees_with_transparent_page_compression.ibd

```
---


## normal
```sql
CREATE TABLE employees (
    emp_no      BIGINT          NOT NULL,
    birth_date  DATE            NOT NULL,
    first_name  VARCHAR(14)     NOT NULL,
    last_name   VARCHAR(16)     NOT NULL,
    gender      ENUM ('M','F')  NOT NULL,
    hire_date   DATE            NOT NULL,
    PRIMARY KEY (emp_no)
);

mysql> select count(*) from employees;
+----------+
| count(*) |
+----------+
|   200000 |
+----------+
1 row in set (0.01 sec)

-rw-r-----. 1 mysql3307 mysql 29M 2月  19 19:17 employees.ibd
```

---