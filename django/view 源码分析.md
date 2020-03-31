## 基于类的视图
最初 django 中的 view 是一个个的函数，从代码复用的角度函数比类要差一些，为了尽可能的复用代码 django 引入了基于类的视图。

为了与之前的代码兼容 View 类提供了一个`as_view`的函数，这个函数执行完成之后会返回一个函数，而被返回的这个函数在参数上与之前的基于函数的 view 的形参是一样的。 所以在 urls.py 中经常可以看到类似代码。

```python
urlpatterns = [
    path('mycnf', mycnf.ConfigGenerateView.as_view(),
         name="onlinetools-mysql-cnf-generate-view")]
```

![cbv](static/2020-14/cbv.png)

google-adsense

---

## as_view 函数做了什么
as_view 的关键逻辑如下。
```python
class View:
    """
    Intentionally simple parent class for all views. Only implements
    dispatch-by-method and simple sanity checking.
    """
    noter = noter.getChild("View")

    http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']

    def __init__(self, **kwargs):

        for key, value in kwargs.items():
            setattr(self, key, value)

    @classonlymethod
    def as_view(cls, **initkwargs):

        for key in initkwargs:

            # key 是 http 的请求方法名就报错
            if key in cls.http_method_names:
                raise TypeError("You tried to pass in the %s method name as a "
                                "keyword argument to %s(). Don't do that."
                                % (key, cls.__name__))

            # 如果 key 和类的属性同名了，也报错
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r. as_view "
                                "only accepts arguments that are already "
                                "attributes of the class." % (cls.__name__, key))

        # 定义 view 函数，用于兼容老版本
        def view(request, *args, **kwargs):

            # 根据 urls.py 中调用 as_view 的参数创建实例
            self = cls(**initkwargs)

            self.setup(request, *args, **kwargs)

            # view 的返回值直接就是 dispatch 执行后的返回值
            return self.dispatch(request, *args, **kwargs)

        return view

    def dispatch(self, request, *args, **kwargs):

        # 根据 http 请求方式的不同，选择与之对应的处理函数
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed

        return handler(request, *args, **kwargs)

```
所以整个调用链就变成了。
```python
as_view -> view -> dispatch ->[get | post | head ... ]
```

---

## 用法
由源码可以看出 dispatch 最终还是要调用其自身的 post,get,head 等待方法，也就是说我们只要实现它们就行了，下面代码实现一个 hello-world 的视图。
```python
from django.urls import path
from django.views.generic.base import View
from django.http.response import JsonResponse


class HelloView(View):
    def get(self, *args, **kwargs):
        """
        """
        return JsonResponse({'message': 'hello-world'})


urlpatterns = [
    path('hello', HelloView.as_view(), name="hello-world"),
]
```

---


## 运行
运行开发服务器。
```bash
python3 manage.py runserver 127.0.0.1:8080
Watching for file changes with StatReloader
Performing system checks...

all keys are valid
System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

March 31, 2020 - 09:31:04
Django version 3.0, using settings 'hp.settings'
Starting development server at http://127.0.0.1:8080/
Quit the server with CONTROL-C.
[31/Mar/2020 09:31:12] "GET /hello HTTP/1.1" 200 26
Not Found: /favicon.ico
[31/Mar/2020 09:31:12] "GET /favicon.ico HTTP/1.1" 404 1987
```
通过 curl 访问。
```bash
curl http://127.0.0.1:8080/hello

{"message": "hello-world"}
```

---


## 项目源码
这个例子的源代码也放在了 github 上 [class-base-view-hello-world](https://github.com/Neeky/class-base-view-hello-world) 。

---