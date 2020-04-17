## 背景
标准库中没有直接用于管理 Linux 用户的库，如果想要用 Python 完成创建用户、删除用户 等等 ...  这样的工作；一个常见的解决方案是在 Python 中调用 `useradd`,`userdel` 这样的命令。

```python
import subprocess

subprocess.run(f"useradd mysql", shell=True)
```

虽然这样做可以达到目的，但是这样做的话代码就太松散了，我想做一个尽可能对 pythoner 友好的 API。

---

## 安装 dbm-agent
`usermange` 模块对用户管理进行了封装，要想使用它好非常简单，直接把 dbm-agent 这个软件包安装上就行。
```bash
pip3 install dbm-agent                                                        
Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple                                
Collecting dbm-agent 

Installing collected packages: dbm-agent
  Running setup.py install for dbm-agent ... done
Successfully installed dbm-agent-0.10.0
```
![uls](static/2020-16/lus.png)

用户管理是 dbm-agent-0.10.0 版本的新功能 ，如果你的版本过低是不支持的。

google-adsense

---

## create_group 创建用户组
使用 python 创建 linux 用户组。
```python
In [1]: from dbma.usermanage import LinuxUsers as lus                                            
                                                                                                 
In [2]: lus.create_group('zabbix')                                                               
```
验证效果。
```bash
cat /etc/group | grep zabbix
zabbix:x:1005:
```

---

## create_user 创建用户
使用 python 创建 linux 用户。
```python
In [1]: from dbma.usermanage import LinuxUsers as lus                                            
                                                                                                 
In [2]: lus.create_user("zabbix")
```
验证效果。
```bash
cat /etc/passwd | grep zabbix                                               
zabbix:x:1005:1005::/home/zabbix:/bin/bash
```

---

## delete_user 删除用户
使用 python 删除 linux 用户。
```python
In [1]: from dbma.usermanage import LinuxUsers as lus                                            
                                                                                                 
In [2]: lus.delete_user("zabbix")
```
验证效果。
```bash
cat /etc/passwd | grep zabbix
# 过滤不到内容了

```

---

## delete_group 删除用户组
使用 python 删除 linux 用户组。
```python
In [1]: from dbma.usermanage import LinuxUsers as lus                                            
                                                                                                 
In [2]: lus.delete_user("zabbix")
```
验证效果。
```bash
cat /etc/group | grep zabbix
# 过滤不到内容了。
```
---

## uid_gid 查询用户的 uid 和 gid 
```python
In [1]: from dbma.usermanage import LinuxUsers as lus  

In [2]: lus.uid_gid("root")                                                                      
Out[2]: (0, 0) 
```

---

## 查询用户和用户组是否存在
```python
In [1]: from dbma.usermanage import LinuxUsers as lus 

In [2]: lus.is_user_exists("root")                                                               
Out[2]: True                                                                                     

In [3]: lus.is_group_exists("root")                                                             
Out[3]: True 
```
---

## 源码

目前 dbm-agent 托管在 [github](https://github.com/Neeky/dbm-agent) 。

---