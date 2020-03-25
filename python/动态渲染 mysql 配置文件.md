## 背景
前段时间写了一个自动化安装 `MySQL` 的程序，其中有一个环节就是动态的渲染 `my.cnf` 文件；总的解决方案就是像 Django 渲染 html 页面一样，用渲染模板的方式来解决。
```ini
[mysqld]
basedir = {{basedir}}
datadir = {{datadir}}
port    = {{port}}
user    = {{user}}
```
![my-cnf-jinja](static/2020-12/my-cnf-jinja.png)
google-adsense

---

## 代码实现
我在渲染引擎的选择上使用了 jinja2 这个模板引擎，去掉其它逻辑一个最小化的代码如下。
```python
#!/usr/bin/env python3

from  jinja2 import Environment,FileSystemLoader
            
def render_mysql_config_file():
    #通过文件系统加载器，加载当前目录下的 my.cnf.jinja 模板文件
    env = Environment(loader=FileSystemLoader(searchpath='./'))
    tmpl = env.get_template('my.cnf.jinja')
    #给要渲染的参数指定值
    cnfs = {
        'basedir': '/usr/local/mysql/',
        'datadir': '/database/mysql/data/3306/',
        'port': 3306,
        'user'; 'mysql3306'
    }

    tmpl.globals=cnfs
    #不保存到 /etc/my.cnf 了，直接输出到 stdout
    print(tmpl.render())

if __name__ == "__main__":
    render_mysql_config_file()
```
运行效果如下
```bash
python3 cnfs.py 
```
```ini
[mysqld]
basedir = /usr/local/mysql/
datadir = /database/mysql/data/3306/
port    = 3306
user    = mysql3306

```

---

## 总结
通过模板引擎渲染 my.cnf 只要专参数就行了，非常的方便。
