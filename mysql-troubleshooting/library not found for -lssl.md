## 安装 mysqlclient 的时候报错
今天把 Mac 上的 Python3.7 升级到 Python-3.8 在安装 mysqlclient 的时候报错了。
```bash
pip3 install mysqlclient 

Running setup.py install for mysqlclient ... error
    ERROR: Command errored out with exit status 1:
     command: /Library/Frameworks/Python.framework/Versions/3.8/bin/python3.8 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-install-zhca7a0b/mysqlclient/setup.py'"'"'; __file__='"'"'/private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-install-zhca7a0b/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-record-l6bzbnij/install-record.txt --single-version-externally-managed --compile
         cwd: /private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-install-zhca7a0b/mysqlclient/
    Complete output (30 lines):
    running install
    running build
    running build_py
    creating build
    creating build/lib.macosx-10.9-x86_64-3.8
    creating build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/__init__.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/_exceptions.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/compat.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/connections.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/converters.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/cursors.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/release.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    copying MySQLdb/times.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb
    creating build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/__init__.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/CLIENT.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/CR.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/ER.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/FIELD_TYPE.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    copying MySQLdb/constants/FLAG.py -> build/lib.macosx-10.9-x86_64-3.8/MySQLdb/constants
    running build_ext
    building 'MySQLdb._mysql' extension
    creating build/temp.macosx-10.9-x86_64-3.8
    creating build/temp.macosx-10.9-x86_64-3.8/MySQLdb
    gcc -Wno-unused-result -Wsign-compare -Wunreachable-code -fno-common -dynamic -DNDEBUG -g -fwrapv -O3 -Wall -arch x86_64 -g -Dversion_info=(1,4,6,'final',0) -D__version__=1.4.6 -I/usr/local/Cellar/mysql/8.0.12/include/mysql -I/Library/Frameworks/Python.framework/Versions/3.8/include/python3.8 -c MySQLdb/_mysql.c -o build/temp.macosx-10.9-x86_64-3.8/MySQLdb/_mysql.o
    gcc -bundle -undefined dynamic_lookup -arch x86_64 -g build/temp.macosx-10.9-x86_64-3.8/MySQLdb/_mysql.o -L/usr/local/Cellar/mysql/8.0.12/lib -lmysqlclient -lssl -lcrypto -o build/lib.macosx-10.9-x86_64-3.8/MySQLdb/_mysql.cpython-38-darwin.so
    ld: library not found for -lssl
    clang: error: linker command failed with exit code 1 (use -v to see invocation)
    error: command 'gcc' failed with exit status 1
    ----------------------------------------
ERROR: Command errored out with exit status 1: /Library/Frameworks/Python.framework/Versions/3.8/bin/python3.8 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-install-zhca7a0b/mysqlclient/setup.py'"'"'; __file__='"'"'/private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-install-zhca7a0b/mysqlclient/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /private/var/folders/w_/_457k4h53vq0p_1k1nzf5cfm0000gn/T/pip-record-l6bzbnij/install-record.txt --single-version-externally-managed --compile Check the logs for full command output.

```

---

## 解析方案
看样子是 openssl 没有安装，通过 brew 把它安装上。
```bash
brew install openssl


Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles/openssl%401.1-1.1.1g.mojave.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/5c/5c9d113393ff3efc95e5509175305fc9304fba35390a61915ed2864941c423f2?__gda__=exp=1591068981~hmac=94221d3c706ff3baa1a4b0b157e0c20dcc07af58c4bcdcae7205ed9aa790
######################################################################## 100.0%
==> Pouring openssl@1.1-1.1.1g.mojave.bottle.tar.gz
==> Caveats
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
  /usr/local/etc/openssl@1.1/certs

and run
  /usr/local/opt/openssl@1.1/bin/c_rehash

openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides LibreSSL.

If you need to have openssl@1.1 first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> /Users/neekyjiang/.bash_profile

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

==> Summary
🍺  /usr/local/Cellar/openssl@1.1/1.1.1g: 8,059 files, 18MB
==> `brew cleanup` has not been run in 30 days, running now...
Removing: /Users/neekyjiang/Library/Caches/Homebrew/descriptions.json... (259.9KB)
Removing: /Users/neekyjiang/Library/Logs/Homebrew/pcre2... (64B)
Removing: /Users/neekyjiang/Library/Logs/Homebrew/git... (64B)
Pruned 0 symbolic links and 2 directories from /usr/local
```
可以看到 brew 给到了我们导出库文件和头文件的命令，我们要执行给到的命令这样 gcc 这可以找到 openssl 的库了。
```bash
export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
```
---

## 验证
```bash
pip3 install mysqlclient numpy pandas
Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple
Collecting mysqlclient
  Downloading https://mirrors.cloud.tencent.com/pypi/packages/d0/97/7326248ac8d5049968bf4ec708a5d3d4806e412a42e74160d7f266a3e03a/mysqlclient-1.4.6.tar.gz (85kB)
     |████████████████████████████████| 92kB 864kB/s 

Installing collected packages: mysqlclient
  Running setup.py install for mysqlclient ... done
Successfully installed mysqlclient-1.4.6 
WARNING: You are using pip version 19.2.3, however version 20.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

---