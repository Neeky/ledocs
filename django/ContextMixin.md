## 概要
在 django class-base-view 中常常会用到 `get_context_data(self, **kwargs)` 这个方法，今天就来看一下这方法是怎么实现的。

![get-conext-data](static/2020-27/get-context-data.jpg)

google-adsense

---

## 源码解读
get-context-data 方法定义在 `django-3.0/django/views/generic/base.py` 文件的 GetContextMinx 这个混入当中。
```python
class ContextMixin:
    extra_context = None

    def get_context_data(self, **kwargs):

        kwargs.setdefault('view', self)
        if self.extra_context is not None:
            kwargs.update(self.extra_context)
        return kwargs
```
可以看到 get_context-data 函数只做了两件事  

1、把 kwargs.view 设置为 self 。 

2、当 extra_context 不等于 None 的时候把 extra_context 中的键值更新到 kwargs 字典当中去。

---

