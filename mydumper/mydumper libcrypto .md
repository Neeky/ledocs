## 背景
![sqlpy](static/2020-29/sqlpy-mydumper.jpg)

编译安装 mydumper 没有问题，但是运行时报错，编译安装的过程如下。
```bash
# ---- ---- ---- ----
cmake .
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Using mysql-config: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysql_config
-- Found MySQL: /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/include, /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/libmysqlclient.so;/usr/lib64/libpthread.so;/usr/lib64/libm.so;/usr/lib64/librt.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libcrypto.so;/usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libssl.so;/usr/lib64/libdl.so
-- Found ZLIB: /usr/lib64/libz.so (found version "1.2.7") 
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.27.1") 
-- checking for one of the modules 'glib-2.0'
-- checking for one of the modules 'gthread-2.0'
-- checking for module 'libpcre'
--   found libpcre, version 8.32
-- Found PCRE: /usr/include  

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
 
# ---- ---- ---- ----
 pwd
/tmp/mydumper-0.9.5

# ---- ---- ---- ----
make && make install                                            
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
[ 62%] Built target mydumper
[100%] Built target myloader
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/mydumper
-- Removed runtime path from "/usr/local/bin/mydumper"
-- Installing: /usr/local/bin/myloader
-- Removed runtime path from "/usr/local/bin/myloader"
```
---

可以看到整个编译安装的过程并没有报错，但是想运行 mydumper 是不行的，它会报错。
```bash
mydumper --help                                                 
mydumper: error while loading shared libraries: libcrypto.so.1.1: cannot open shared object file: No such file or directory

ldconfig
ldconfig: 文件 /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libcrypto.so 己被截断
ldconfig: 文件 /usr/local/mysql-8.0.20-linux-glibc2.12-x86_64/lib/private/libcrypto.so.1.1 己被截断
```

---


## 分析
说实话，这个问题困扰了我好几天，突然来了一下灵感，要对对比一下我执行文件是不是有问题。可以看到两个可执行文件的 md5 值完成对不上。

```bash
md5sum /usr/local/bin/mydumper
e10796e5e67b76891b274132dbb3f046  /usr/local/bin/mydumper

md5sum ./mydumper                                               
64979005e846155e359bac41891e0584  ./mydumper
```
---

应该是 `make install ` 把 ./mydumper 和 ./myloader 复制到 /usr/local/bin 的时候了一问题，我手动的复制过去。

```bash
mv mydumper /usr/local/bin                                      
mv：是否覆盖"/usr/local/bin/mydumper"？ y


mv myloader /usr/local/bin/                                     
mv：是否覆盖"/usr/local/bin/myloader"？ y
```
---

现在没有问题了
```bash
mydumper --help                                                                           
Usage:
  mydumper [OPTION?] multi-threaded MySQL dumping

Help Options:
  -?, --help                  Show help options

Application Options:
  -B, --database              Database to dump
...
...
...
```

---