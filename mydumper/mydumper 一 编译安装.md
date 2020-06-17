## 背景
我的上一份工作是做 MySQL 技术支持的，我和服务对象多数是银行、电信；他们的系统都求一个稳字。在数据库备份这一块多用的是 mysqlbackup 这个企业级的备份工具。

多年的相处 mysqlbackup 给了一个非常好的映像，慢慢的就从路人转成了粉丝。然而多数据互联网公司在数据库上都不想花钱，更不要说把钱花在一个备份软件上了，如果遇到了这种情况 mydumper 是一个不错的选择。

![sqlpy](static/2020-25/sqlpy-mydumper-01.jpg)

---

## 官方项目路径
[mydumper](https://github.com/maxbube/mydumper) 的源代码放在了 github 上，文本介绍源码安装。[源码](https://github.com/maxbube/mydumper/releases) 可以在这里下载。

---

## 第一步 安装依赖
mydumper 是 C 语言写的，所以 C 语言的编译环境要有。
```bash
yum -y install cmake gcc gcc-c++ glib2-devel
```
如果你的主机上还没有安装过 MySQL 推荐使用 `dbm-agent` 安装 MySQL，它会帮你导出若干环境变量和配置。如果你不想的话可以用
```bash
yum -y install mysql-devel
```

---

## 第二步 下载源码包并解压
下载源码并解压到 /tmp/
```
cd /tmp/
wget https://codeload.github.com/maxbube/mydumper/tar.gz/v0.9.5

tar -xvf mydumper-0.9.5.tar.gz -C ./
cd mydumper-0.9.5
```

google-adsense

---

## 第三步 编译安装

```bash
# 3.1 cmake .
cmake .
-- Using mysql-config: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysql_config
-- Found MySQL: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/include, /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/libmysqlclient.so;/usr/lib64/libpthread.so;/usr/lib64/libm.so;/usr/lib64/librt.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libcrypto.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libssl.so;/usr/lib64/libdl.so
-- checking for one of the modules 'glib-2.0'
-- checking for one of the modules 'gthread-2.0'

CMake Warning at docs/CMakeLists.txt:9 (message):
  Unable to find Sphinx documentation generator


-- ------------------------------------------------
-- MYSQL_CONFIG = /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysql_config
-- CMAKE_INSTALL_PREFIX = /usr/local
-- BUILD_DOCS = ON
-- WITH_BINLOG = OFF
-- RUN_CPPCHECK = OFF
-- Change a values with: cmake -D<Variable>=<Value>
-- ------------------------------------------------
-- 
-- Configuring done
-- Generating done
-- Build files have been written to: /tmp/mydumper-0.9.5

# 3.2 make
make                                                               
Scanning dependencies of target mydumper
[ 12%] Building C object CMakeFiles/mydumper.dir/mydumper.c.o
[ 25%] Building C object CMakeFiles/mydumper.dir/server_detect.c.o
[ 37%] Building C object CMakeFiles/mydumper.dir/g_unix_signal.c.o
[ 50%] Building C object CMakeFiles/mydumper.dir/connection.c.o
[ 62%] Building C object CMakeFiles/mydumper.dir/getPassword.c.o
Linking C executable mydumper
[ 62%] Built target mydumper
Scanning dependencies of target myloader
[ 75%] Building C object CMakeFiles/myloader.dir/myloader.c.o
[ 87%] Building C object CMakeFiles/myloader.dir/connection.c.o
[100%] Building C object CMakeFiles/myloader.dir/getPassword.c.o
Linking C executable myloader
[100%] Built target myloader

# 3.3 make install
make install
[ 62%] Built target mydumper
[100%] Built target myloader
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/mydumper
-- Removed runtime path from "/usr/local/bin/mydumper"
-- Installing: /usr/local/bin/myloader
-- Removed runtime path from "/usr/local/bin/myloader"
```
就这么简单的三步就完成了。

---

## 检查
如果能执行 mydumper 命令说明安装成功了。
```bash
mydumper --help                                                               
Usage:
  mydumper [OPTION?] multi-threaded MySQL dumping

Help Options:
  -?, --help                  Show help options

Application Options:
  -B, --database              Database to dump
  -T, --tables-list           Comma delimited table list to dump (does not exclude regex option)
  -O, --omit-from-file        File containing a list of database.table entries to skip, one per line (skips before applying regex option)

... ...
```
---

## 其它问题
一个比较常见的问题是忘记了安装 `glib2-devel`，如果缺少它 cmake 的时候就报这样的错。
```bash
cmake .
-- Using mysql-config: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysql_config
-- Found MySQL: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/include, /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/libmysqlclient.so;/usr/lib64/libpthread.so;/usr/lib64/libm.so;/usr/lib64/librt.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libcrypto.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libssl.so;/usr/lib64/libdl.so
-- checking for one of the modules 'glib-2.0'
CMake Error at /usr/share/cmake/Modules/FindPkgConfig.cmake:363 (message):
  None of the required 'glib-2.0' found
Call Stack (most recent call first):
  cmake/modules/FindGLIB2.cmake:10 (pkg_search_module)
  CMakeLists.txt:10 (find_package)


-- checking for one of the modules 'gthread-2.0'
CMake Error at /usr/share/cmake/Modules/FindPkgConfig.cmake:363 (message):
  None of the required 'gthread-2.0' found
Call Stack (most recent call first):
  cmake/modules/FindGLIB2.cmake:11 (pkg_search_module)
  CMakeLists.txt:10 (find_package)



CMake Warning at docs/CMakeLists.txt:9 (message):
  Unable to find Sphinx documentation generator


-- ------------------------------------------------
-- MYSQL_CONFIG = /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysql_config
-- CMAKE_INSTALL_PREFIX = /usr/local
-- BUILD_DOCS = ON
-- WITH_BINLOG = OFF
-- RUN_CPPCHECK = OFF
-- Change a values with: cmake -D<Variable>=<Value>
-- ------------------------------------------------
-- 
CMake Error: The following variables are used in this project, but they are set to NOTFOUND.
Please set them or make sure they are set and tested correctly in the CMake files:
GLIB2_LIBRARIES (ADVANCED)
    linked by target "mydumper" in directory /tmp/mydumper-0.9.5
    linked by target "myloader" in directory /tmp/mydumper-0.9.5
GTHREAD2_LIBRARIES (ADVANCED)
    linked by target "mydumper" in directory /tmp/mydumper-0.9.5
    linked by target "myloader" in directory /tmp/mydumper-0.9.5

-- Configuring incomplete, errors occurred!
See also "/tmp/mydumper-0.9.5/CMakeFiles/CMakeOutput.log".
```

---