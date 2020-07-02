## 概要
django 中所有的基于类的视图都继承自 `django.views.generic.base.View`，那它是怎么实现的呢？又是怎么做到与之前的基于函数的视图兼容的呢？

![sqlpy](static/2020-27/sqlpy-django-view.jpg)



---

## View 的使用方式
在深入源代码之前我们先来看一下，View 在实际项目中的使用方式，下面使用基于类的视图简单的实现输出 `hello world.` 的功能。
```python
class HelloWorldView(View):
    """
    """

    def get(self, request, *args, **kwargs):
        return JsonResponse({
            'message': 'hello-world.'
        })


urlpatterns = [
    path('', HelloWorldView.as_view(), name='hello-world-view'),
]

```

页面的效果如下。

![sqlpy](static/2020-27/sqlpy-view.jpg)

---

## View 执行流程
老版本的视图是通过函数来实现的，因为函数的代码重用能力和类并不在同一个级别，为了可以重用更多的代码 django 开始支持基于类的视图。那它又是怎么兼容之前老的代码的呢？

答案是通过 `View.as_view()` 这个函数把自己变得看上去和之前一样。

```python
# 简化版本的主线逻辑是这样的。
class View:
    @classonlymethod
    def as_view(cls, **initkwargs):

        def view(request, *args, **kwargs):
            # 创建实例
            self = cls(**initkwargs)
            if hasattr(self, 'get') and not hasattr(self, 'head'):
                self.head = self.get

            self.setup(request, *args, **kwargs)

            if not hasattr(self, 'request'):
                raise AttributeError(
                    "%s instance has no 'request' attribute. Did you override "
                    "setup() and forget to call super()?" % cls.__name__
                )
            return self.dispatch(request, *args, **kwargs)

        return view
```
可以看到 as_view 这个函数调用之后返回的 view 在形式参数上是与之前的基于函数的视图是一样的，这样就达到了兼容的目的。

另一个就是 view 的每一次执行都会创建一个 View 的实例，也就是说每一个 http 请求最后干活的就是一个 View 的实例，哪 http 请求又是怎么对应的实例的方法的呢？这个就要看 
View.dispatch 方法了。

---


## View.dispatch 方法分析
关键代码如下。
```python
class View:
    http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']

    def dispatch(self, request, *args, **kwargs):
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed

        return handler(request, *args, **kwargs)
```

可以看到 dispatch 把 Http 请求方法(method)与 View 的方法(function) 对应上了。

---