## 背景

用 Python 已经有好几年了，从之前乱来(不写文档字符串)到给自己的代码写上一段自己都看不懂的说明，到现在看 numpy 的源代码并被它的风格所吸引，最后在它的基础之上改进得到了
自己的认为比较好的实践。

现在把经验总结一下，假设我们要实现一下加法函数，什么都不写的情况下它是这样的。

```python
def add(x, y):
    return x + y
```

像这种简短的函数不写注释其实还好，如果长一点不写注释就有点要人命了，如果整个项目都没有注释就完完了。

![sqlpy](static/2020-29/sqlpy-comments.jpg)

---

## 第一条 加上类型提示
python 已经支持类型提示了，类型提示可以看成是注释的另一种形式，加上类型提示的另一个好处是可以方便自己，比如 vscode 就可以根据类型提示给出我们更多的智能提示。

```python
def add(x:float=0,y:float=0)->float:
    return x + y
```

这样就可以明显的看出函数接收一个实数并返回一个实数，而不是 str 这个类型的参数(str也支持 `+` 运算符)。

---

## 第二条 加上功能描述
要在文档字符串中写上自己是干什么的。
```python
def add(x: float = 0, y: float = 0) -> float:
    """
    实现两个实例的加法
    """
    return x + y
```
---

## 第三条 加上参数描述
参数只有类型提示还不足够，最好要对单个参数的详细说明。按照 numpy 的风格来说参数描述用 `Parameters` 段，后面是一条下划线，然后是参数名和类型说明，缩进后的部分是参数的描述。

```python
def add(x: float = 0, y: float = 0) -> float:
    """
    实现两个实例的加法

    Parameters
    ----------
    x: int | folat 类型的值
        被加数
    
    y: int | folat 类型的值
        加数

    """
    return x + y
```
---

## 第四条 加上返回值的描述
返回值的描述通过 `Returns` 段来完。
```python
def add(x: float = 0, y: float = 0) -> float:
    """
    实现两个实例的加法

    Parameters
    ----------
    x: int | folat 类型的值
        被加数
    
    y: int | folat 类型的值
        加数

    Retruns
    -------
    result: int

    """
    return x + y

```

---

## 第五条 加上异常的描述
如果有抛出异常的代码，那么可以用 `Raises` 段来描述，
```python
def add(x: float = 0, y: float = 0) -> float:
    """
    实现两个实例的加法

    Parameters
    ----------
    x: int | folat 类型的值
        被加数

    y: int | folat 类型的值
        加数

    Retruns
    -------
    result: int

    Raises
    ------
    TypeError:
        当参数的类型不正确时(x 和 y 不能执行 + 法运算)

    """
    return x + y
```
---

## 第六条 加上测试用例
加测试用例一来可以检查代码逻辑上的正确性，二来还可以作为其它上使用我们代码时的一个例子，默认使用 `Examples` 段来描述。
```python

def add(x: float = 0, y: float = 0) -> float:
    """
    实现两个实例的加法

    Parameters
    ----------
    x: int | folat 类型的值
        被加数

    y: int | folat 类型的值
        加数

    Retruns
    -------
    result: int

    Raises
    ------
    TypeError:
        当参数的类型不正确时(x 和 y 不能执行 + 法运算)

    Examples:
    --------
    >>> add(1,1)
    2
    >>> add(-1,-1)
    -2
    >>> add(0,0)
    0
    >>> add(-1,1)
    0
    """
    return x + y

```

---

## 其它

如果你还有其它要说明的可以放在 `Notes` 和 `See Also` 这两个段中。

---