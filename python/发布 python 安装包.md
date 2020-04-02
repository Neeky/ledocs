## 概要
还记得 Python 可以用 pip install 来安装依赖包吗？那这些包是怎么制作出来的呢？下面以发布 week-of-year 这个软件包为例，来讲一下它的详细制作过程。

![week-of-year](static/2020-14/week-of-year.png)

google-adsense

---

## week-of-year
为了让我们的项目尽可能的有意义，不要让它成为 PyPI 上的垃圾，我会在这个包中实现一个 `week-of-year` 的命令，这个命令会打印当前是今年的第几周。

---

## 第一步 创建项目
创建这个项目的目录树，最终目录的
```bash
tree .
├── README.md
├── bin
│   └── week-of-year
├── setup.py
└── wofy
    ├── __init__.py
    └── utils.py

```
整个项目的源码已经开源在 github ，所以这里就不对功能上的实现做过多的展开，这里会把主要精力用在如何制作发行包的说明上。

---


## 第二步 配置项目
整个软件包的配置是在 setup.py 这个文件中定义的，下面来介绍一下这个文件。
```python
from setuptools import setup


setup(name='week-of-year',                    #软件包的名字
      version='0.0.1',                        #版本
      description='返回当前是今年的第几周',       #描述信息
      author="Neeky",                         #作者
      author_email="neeky@live.com",          #邮件
      maintainer='Neeky',                     #作者
      maintainer_email='neeky@live.com',      #邮件
      scripts=['bin/week-of-year'],           #软件也要导出的命令
      packages=['wofy'],                      #软件包要导出的“包”
      url='https://github.com/Neeky/wofy',    #项目地址
      python_requires='>=3.6.*',              #版本要求
      classifiers=[
          'Development Status :: 4 - Beta',
          'Intended Audience :: Developers',
          'Operating System :: POSIX',
          'Operating System :: MacOS :: MacOS X',
          'Programming Language :: Python :: 3.6',
          'Programming Language :: Python :: 3.7',
          'Programming Language :: Python :: 3.8']
      )
```
事实上这个 setup.py 也不要死记，`ctrl+c ctrl+v`然后按自己的需改一改就行了。

---

## 第三步 生成安装包
生成源码安装包
```bash
python3 setup.py sdist


running sdist
running egg_info
writing week_of_year.egg-info/PKG-INFO
writing dependency_links to week_of_year.egg-info/dependency_links.txt
writing top-level names to week_of_year.egg-info/top_level.txt
reading manifest file 'week_of_year.egg-info/SOURCES.txt'
writing manifest file 'week_of_year.egg-info/SOURCES.txt'
running check
creating week-of-year-0.0.1
creating week-of-year-0.0.1/bin
creating week-of-year-0.0.1/week_of_year.egg-info
creating week-of-year-0.0.1/wofy
copying files to week-of-year-0.0.1...
copying README.md -> week-of-year-0.0.1
copying setup.py -> week-of-year-0.0.1
copying bin/week-of-year -> week-of-year-0.0.1/bin
copying week_of_year.egg-info/PKG-INFO -> week-of-year-0.0.1/week_of_year.egg-info
copying week_of_year.egg-info/SOURCES.txt -> week-of-year-0.0.1/week_of_year.egg-info
copying week_of_year.egg-info/dependency_links.txt -> week-of-year-0.0.1/week_of_year.egg-info
copying week_of_year.egg-info/top_level.txt -> week-of-year-0.0.1/week_of_year.egg-info
copying wofy/__init__.py -> week-of-year-0.0.1/wofy
copying wofy/utils.py -> week-of-year-0.0.1/wofy
Writing week-of-year-0.0.1/setup.cfg
Creating tar archive
removing 'week-of-year-0.0.1' (and everything under it)
```
执行完成之后就可以在 dist 目录下看到软件包了。
```bash
ll dist
total 8
-rw-r--r--  1 jianglexing  staff  1576  4  2 16:57 week-of-year-0.0.1.tar.gz
```

---

## 第四步 验证
通过在本机上安装一下看能否成功来验证，安装包是否可用。
```bash
pip3 install dist/week-of-year-0.0.1.tar.gz

Successfully installed week-of-year-0.0.1
```
执行一下命令看功能上是否正确
```bash
week-of-year 
2020-4
```
功能上没有问题，看来安装包是可用的。

---

## 第五步 上传包到 PyPI
通过 `twine` 命令上传包到 PyPI，如果你还没有 PyPI 账号要注册哦。
```bash
twine upload dist/week-of-year-0.0.1.tar.gz 

Uploading distributions to https://upload.pypi.org/legacy/
Uploading week-of-year-0.0.1.tar.gz
100%|██████████████████████████████████████| 5.04k/5.04k [00:00<00:00, 5.54kB/s]
```
完成之后在 PyPI 就可以找到了[week-of-year](https://pypi.org/project/week-of-year/) 。

---

## 通过 pip 安装
在把包上传到 PyPI 之后就可以通过 pip3 来安装了。
```bash
pip3 install week-of-year

Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple
Collecting week-of-year
  Downloading https://mirrors.cloud.tencent.com/pypi/packages/b1/6b/c28daf613e3b2de4c64f45428da521bcfd8652e0a95f4e0b77a272275986/week-of-year-0.0.1.tar.gz
Installing collected packages: week-of-year
  Running setup.py install for week-of-year ... done
Successfully installed week-of-year-0.0.1
```

---









