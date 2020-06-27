## 概要
如何得到 django 对数据库的操作日志呢？

![django-log](static/2020-27/django-log.jpg)

google-adsense

---

## MySQL 场景
如果配置中使用的数据库是 MySQL，像下面的配置这样；
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dbloger',
        'USER': 'dbloger',
        'PASSWORD': 'dbloger',
        'HOST': '127.0.0.1',
        'PORT': 3306
    }
}
```
这种情况下可以直接打开 MySQL 的 general-log ，因为应用程序的所有操作都会在这里记日志，所以打开它可以看到 django 的操作日志。
```sql
-- 打开 general-log
mysql> set @@global.general_log=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'general%';
+------------------+--------------------------------------------------+
| Variable_name    | Value                                            |
+------------------+--------------------------------------------------+
| general_log      | ON                                               |
| general_log_file | /usr/local/homebrew/var/mysql/NEEKYJIANG-MB1.log |
+------------------+--------------------------------------------------+
2 rows in set (0.00 sec)
```
应用程序有操作数据库的话，就能看到日志。
```
2020-04-08T07:31:55.511039Z	  927 Connect	dbloger@localhost on dbloger using SSL/TLS
2020-04-08T07:31:55.511881Z	  927 Query	set autocommit=0
2020-04-08T07:31:55.513629Z	  927 Query	set autocommit=1
2020-04-08T07:31:55.514672Z	  927 Query	SELECT @@SQL_AUTO_IS_NULL
2020-04-08T07:31:55.515687Z	  927 Query	SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
2020-04-08T07:31:55.516514Z	  927 Query	INSERT INTO `foo_visitlogmodel` (`visit_at`, `http_url`) VALUES ('2020-04-08 07:31:55.516245', '/foo/')
2020-04-08T07:31:55.521283Z	  927 Quit
```

---

## 当前方案存在的问题
前面介绍的方案只适用于 MySQL 作为后台数据库的场景，为了能适用更多的场景，最好还要让 django 主动的打印日志。

---

## 所有场景
想要 django 主动的打印数据库操作相关的日志，就是要配置`django.db.backends` 这个 logger ，关键部分如下。
```python
'loggers': {
    'django.db.backends': {
        'level': 'DEBUG',
        'handlers': ['db-handler']
    }
}
```
完整的日志配置如下。
```python
# 配置日志
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'simple': {
            'format': '{asctime} {levelname} {name} {message}',
            'style': '{',
        },
    },
    'handlers': {
        # 项目日志
        'dbloger-handler': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        },
        # 数据库日志
        'db-handler': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        }
    },
    'loggers': {
        'dbloger': {
            'level': 'INFO',
            'handlers': ['dbloger-handler'],
            'propagate': True,
        },
        'django.db.backends': {
            'level': 'DEBUG',
            'handlers': ['db-handler']
        },
    }
}

```
当执行数据库操作时就可以看到日志了。
```
2020-04-08 07:31:55,515 DEBUG django.db.backends (0.001) SELECT @@SQL_AUTO_IS_NULL; args=None
2020-04-08 07:31:55,515 DEBUG django.db.backends (0.000) SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED; args=None
2020-04-08 07:31:55,519 DEBUG django.db.backends (0.003) INSERT INTO `foo_visitlogmodel` (`visit_at`, `http_url`) VALUES ('2020-04-08 07:31:55.516245', '/foo/'); args=['2020-04-08 07:31:55.516245', '/foo/']
```

---

## 项目源码
整个项目开源在了 github [dbloger](https://github.com/Neeky/leorg) 启动服务后 `curl http://127.0.0.1:8080/foo/` 可以触发数据库操作。

---





