## 概要
今天手工编译安装 mysql-connector-python-8.0.20 的时候遇到了问题。
```bash
python3 setup sdist
pip3 install dist/mysql-connector-python-8.0.20.tar.gz
```
报错信息如下。
```bash
ERROR: Command errored out with exit status 1:
 command: /Library/Frameworks/Python.framework/Versions/3.8/bin/python3.8 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/private/tmp/pip-req-build-_v5x29xh/setup.py'"'"'; __file__='"'"'/private/tmp/pip-req-build-_v5x29xh/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /private/tmp/pip-record-gto2n9ru/install-record.txt --single-version-externally-managed --compile --install-headers /Library/Frameworks/Python.framework/Versions/3.8/include/python3.8/mysql-connector-python
     cwd: /private/tmp/pip-req-build-_v5x29xh/
Complete output (6 lines):
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help

error: option --single-version-externally-managed not recognized
----------------------------------------
```

---

## 分析
看到报错的信息是选项不支持，通常来说只会在新版本上引入新的选项；难道是我的 python 安装工具包版本太低了？于是先升级。
```bash
pip3 install setuptools --upgrade
pip3 install wheel --upgrade
```
---

## 再次安装
升级安装工具后再次安装成功。
```bash
pip3 install dist/mysql-connector-python-8.0.20.tar.gz
```
输出如下。
```bash
Installing collected packages: mysql-connector-python
  Attempting uninstall: mysql-connector-python
    Found existing installation: mysql-connector-python 8.0.18
    Uninstalling mysql-connector-python-8.0.18:
      Successfully uninstalled mysql-connector-python-8.0.18
Successfully installed mysql-connector-python-8.0.20
```

---

