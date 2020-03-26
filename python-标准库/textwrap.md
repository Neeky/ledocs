## 概要
textwrap 这个库主要作用是对文本进行各种各样的美化，让它在字符页面中尽可能的好看。

![shorten](static/2020-13/shorten.png)
google-adsense

---

## 截取 shorten
截取一段文字的前 n 个字符，同时它也会尽可能的保证单词的完整性；下面代码用来截取前 60 个字符。
```python
>>> import textwrap
>>> s="""This manual describes features that are not included in every edition of MySQL 8.0; such features may not be included in the edition of MySQL 8.0 licensed to you. If you have any questions about the features included in your edition of MySQL 8.0, refer to your MySQL 8.0 license agreement or contact your Oracle sales representative."""                   
>>> st = textwrap.shorten(s,width=60)
>>> print(len(st))
58
>>> print(st)
This manual describes features that are not included [...]
```

---

## 缩进 indent
用于在行首添加空格，达到缩进的效果，下面的代码实现行首缩进四个空格。
```python
>>> import textwrap
>>> s="This manual describes features that are not included in every edition of MySQL 8.0;"
>>> print(textwrap.indent(s,"    "))
    This manual describes features that are not included in every edition of MySQL 8.0;

#如果要看上去新是在 MySQL 命令行中也非常简单
>>> print(textwrap.indent(s,"mysql> "))
mysql> This manual describes features that are not included in every edition of MySQL 8.0;
```
---

## 去掉缩进 dedent
去掉行首的空格，客观上达到了一种去掉缩进的效果，下面代码会把行首的四个空格去掉。
```python
>>> import textwrap
>>> s="    This manual describes features"
>>> print(textwrap.dedent(s))
This manual describes features
```

---

## 以给定的宽度输出段落 fill
下面代码让段落以 48 字符的宽度输出。
```python
>>> import textwrap
>>> s="""This manual describes features that are not included in every edition of MySQL 8.0; such features may not be included in the edition of MySQL 8.0 licensed to you. If you have any questions about the features included in your edition of MySQL 8.0, refer to your MySQL 8.0 license agreement or contact your Oracle sales representative."""
>>> print(textwrap.fill(s,48))
This manual describes features that are not
included in every edition of MySQL 8.0; such
features may not be included in the edition of
MySQL 8.0 licensed to you. If you have any
questions about the features included in your
edition of MySQL 8.0, refer to your MySQL 8.0
license agreement or contact your Oracle sales
representative.
```
---

## 段落转换成列表 wrap
wrap 会把一段给定的文字转换成字符串列表，一方面它会尽可能的保证单词的完整性，另一方面它还会让各个字符串尽可能的差不多长。
```python
>>> import textwrap
>>> s="""This manual describes features that are not included in every edition of MySQL 8.0; such features may not be included in the edition of MySQL 8.0 licensed to you. If you have any questions about the features included in your edition of MySQL 8.0, refer to your MySQL 8.0 license agreement or contact your Oracle sales representative."""
>>> textwrap.wrap(s)
['This manual describes features that are not included in every edition', 'of MySQL 8.0; such features may not be included in the edition of', 'MySQL 8.0 licensed to you. If you have any questions about the', 'features included in your edition of MySQL 8.0, refer to your MySQL', '8.0 license agreement or contact your Oracle sales representative.']
```

---
