## 老版本的重定向写法

Django 对于重定向有一个专门的实现 `RedirectView`；在正式的讲解 RedirectView 之前先来看一下老的写法。

当用户访问 `/home` 的时候重定向到 `/about` 示例如下。
```python
class HomeView(View):
    """home 页面(当前访问这个页面的时候会被直接重定向到 about 页面)
    """

    def get(self, request, *args, **kwargs):
        """
        """
        # 重定向到 /about 页面
        return HttpResponseRedirect("/about")


class AboutView(View):
    """about 页面
    """

    def get(self, request, *args, **kwargs):
        """
        """
        return JsonResponse({
            'message': 'this-is-about-view.'
        })


urlpatterns = [
    path('home', HomeView.as_view(), name='home-view'),
    path('about', AboutView.as_view(), name='about-view')
]

```

![sqlpy](static/2020-28/sqlpy-redirector-view.jpg)

google-adsense

---

## 改进老版本
老版本存在的问题之一就是它硬编码了 url `/home` ，要改进这个也不难，`django.urls.reverse` pattern_name 转换成对应的 url 地址。
```python
In [1]: from django.urls import reverse 

In [2]: reverse('about-view')                                                   
Out[2]: '/about'
```
对于之前的代码就可以这样改进。
```python
class HomeView(View):
    """home 页面(当前访问这个页面的时候会被直接重定向到 about 页面)
    """

    def get(self, request, *args, **kwargs):
        """
        """
        # 重定向到 /about 页面
        return HttpResponseRedirect(reverse('about-view'))


class AboutView(View):
    """about 页面
    """

    def get(self, request, *args, **kwargs):
        """
        """
        return JsonResponse({
            'message': 'this-is-about-view.'
        })


urlpatterns = [
    path('home', HomeView.as_view(), name='home-view'),
    path('about', AboutView.as_view(), name='about-view')
]

```

---

## 使用 RedirectView 实现
上的例子我们还是用代码实现了部分逻辑，对于重定向这样的通用功能最好是使用配置，RedirectView 就为我们提供了这样的途径。
```python
from django.views.generic.base import RedirectView

class HomeView(RedirectView):
    """
    """
    pattern_name = 'about-view'


class AboutView(View):
    """about 页面
    """

    def get(self, request, *args, **kwargs):
        """
        """
        return JsonResponse({
            'message': 'this-is-about-view.'
        })


urlpatterns = [
    path('home', HomeView.as_view(), name='home-view'),
    path('about', AboutView.as_view(), name='about-view')
]

```

基于 RedirectView 我们通过配置 `pattern_name` 这样属性就实现了功能。

---

## RedirectView 的源码实现
官方的代码把目标 `url` 的计算过程通过 `get_redirect_url` 统一的实现了，并且把所有的 http 请求方法都委托给了 get 。

```python
class RedirectView(View):
    """Provide a redirect on any GET request."""
    permanent = False
    url = None
    pattern_name = None
    query_string = False

    def get_redirect_url(self, *args, **kwargs):
        if self.url:
            url = self.url % kwargs
        elif self.pattern_name:
            url = reverse(self.pattern_name, args=args, kwargs=kwargs)
        else:
            return None

        args = self.request.META.get('QUERY_STRING', '')
        if args and self.query_string:
            url = "%s?%s" % (url, args)
        return url

    def get(self, request, *args, **kwargs):
        url = self.get_redirect_url(*args, **kwargs)
        if url:
            if self.permanent:
                return HttpResponsePermanentRedirect(url)
            else:
                return HttpResponseRedirect(url)
        else:
            logger.warning(
                'Gone: %s', request.path,
                extra={'status_code': 410, 'request': request}
            )
            return HttpResponseGone()

    def head(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)

    def options(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.get(request, *args, **kwargs)
```

---