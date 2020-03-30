## 概要
在 python-3.9.0 之前 python 程序员可以通过如下代码来合并两个字典，生成一个新的字典。
```python
import copy

def main():
    x = {'hello': 'word'}
    y = {'name': 'tim'}
    print(f"id(x) = {id(x)}  id(y)={id(y)}")
    z = copy.copy(x)
    z.update(y)
    print(f"id(z) = {id(z)}   z = {z}")
    #print(f"id(z) = {id(z))} z = {z}") 

if __name__ == "__main__":
    main()
```
运行效果
```bash
python3 old-style.py 
id(x) = 140159953468736  id(y)=140159953468800
id(z) = 140159825804672   z = {'hello': 'word', 'name': 'tim'}
```

![old-style](static/2020-14/old-style.png)

---

## 看看我们都干了什么
想想我们刚才做了什么，我们只是想合并两个字典居然写了三行代码。

第一步 `import copy`

第二步`z = copy.copy(x)` 

第三步`z.update(y)`

当我看到 python-3.9.0 的新语法后，整个人都不好了，python 程序员中真是没有最懒只有更懒！详情可以查询 [PEP-584](https://www.python.org/dev/peps/pep-0584/)。


---

## python-3.9.0 打开新世界
如果使用 python-3.9.0 的新语法的话，一切会简单不少。
```python
def main():
    x = {'hello':'world'}
    y = {'name':'tim'}
    print(f"id(x) = {id(x)}  id(y) = {id(y)}")
    z = x | y #新语法
    print(f"id(z) = {id(z)} z = {z}")


if __name__ == "__main__":
    main()
```
运行情况
```bash
/usr/local/python-3.9.0/bin/python3 main.py                                 
id(x) = 140649444801152  id(y) = 140649444801344
id(z) = 140649444801728 z = {'hello': 'world', 'name': 'tim'}
```

---

## 总结
python-3.9.0 把之前的三行代码变成了现在的一行代码。

---