## 介绍
`python-markdown` 这个库可以把 markdown 转化为 html ，用起来方便，有许多的第三方拓展，再者自己加一些自己的功能也非常方便。

![markdown](static/2020-15/markdown.png)

google-adsense

---


## 安装
[Markdown](https://pypi.org/project/Markdown/) 可以通过 pip 直接安装。 
```bash
pip3 install Markdown

Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple
Collecting Markdown
  Downloading https://mirrors.cloud.tencent.com/pypi/packages/ab/c4/ba46d44855e6eb1770a12edace5a165a0c6de13349f592b9036257f3c3d3/Markdown-3.2.1-py2.py3-none-any.whl (88kB)
     |████████████████████████████████| 92kB 539kB/s                                             
Requirement already satisfied: setuptools>=36 in /usr/local/python-3.8.2/lib/python3.8/site-packages (from Markdown) (41.2.0)
Installing collected packages: Markdown
Successfully installed Markdown-3.2.1
WARNING: You are using pip version 19.2.3, however version 20.0.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

---

## 基本用法
转化一段简单的 markdown 字符串为例子。

```python
import markdown
s = "## hell-world"
print(markdown.markdown(s))
```
运行效果如下。
```bash
python3 main.py

'<h2>hell-world</h2>'
```

API 就是这么的人性化，只要把要转化的 markdown 字符串传递给 `markdown.markdown`函数就行。

---

## 生态
markdown 这个库的生态比较好，一些常用的功能它自己就解决，实在解决不了的还有官方拓展可用。通常来说对于“段落”，“标题”这些简单的元素我们用不到拓展，但是对于“目录”，“代码块” 这些复杂点的东西就要用到拓展才能实现解析了，下面会介绍一些常用的拓展和编写自己的拓展。

google-adsense

---

## markdown.extensions.toc
在上面的例子中，我们看到 markdown 虽然格式化了标题(h2)但是不没能自动生成目录，`markdown.extensions.toc`就能自动为文章的标题生成目录。
```python
# 给 markdown 加上 [TOC] 标记
s="""[TOC]
## python
hello-python

---

## sql
hello-sql

---
"""

# 在处理 markdown 的时候加上 TOC 专用的拓展
print(markdown.markdown(s,extensions=['markdown.extensions.toc']))

```
运行后的输出如下。
```html
<div class="toc">
    <ul>
        <li><a href="#python">python</a></li>
        <li><a href="#sql">sql</a></li>
    </ul>
</div>

<h2 id="python">python</h2>
<p>hello-python</p>
<hr />

<h2 id="sql">sql</h2>
<p>hello-sql</p>
<hr />
```
看只是加了一个简单的 `extensions=['markdown.extensions.toc']` 就实现了目录功能。

---


## markdown.extensions.fenced_code
markdown.extensions.fenced_code 为 markdown 加上格式化代码的功能。
```python
s="""
# 请把 . 号换成 ` 号，这里不方便书写
...sql
select 1 as a;
...
"""
print(markdown.markdown(s,extensions=['markdown.extensions.toc','markdown.extensions.fenced_code']))
```
输出如下。
```bash
python3 main.py

<pre><code class="sql">select 1 as a;
</code></pre>
```
---

## markdown.extensions.tables
`markdown.extensions.tables` 可以用来解析表格。
```python
s="""|**name**|**age**|
|---|---|
|tim| 16|
|tom| 17|
"""
print(markdown.markdown(s,extensions=['markdown.extensions.toc','markdown.extensions.fenced_code','markdown.extensions.tables']))
```
输出如下。
```bash
python3 main.py

<table>
<thead>
<tr>
<th><strong>name</strong></th>
<th><strong>age</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>tim</td>
<td>16</td>
</tr>
<tr>
<td>tom</td>
<td>17</td>
</tr>
</tbody>
</table>
```

---

## 编写自己的拓展
可以看到`markdown.extensions.tables`解析出来的 html 是不带样式的，那怎么加上样式呢？我们可以自定义拓展呀。
```python
from markdown import extensions
from markdown.treeprocessors import Treeprocessor


class BootstrapTreeprocessor(Treeprocessor):
    """
    """

    def run(self, node):
        for child in node.getiterator():
            # 如果是 table
            if child.tag == 'table':
                child.set("class", "table table-bordered table-dark")
            elif child.tag == 'h2':
                child.set("class", "h5 text-secondary mb-4")
            # elif child.tag == 'img':
            #    child.set("class","img-fluid")
        return node


class BootStrapExtension(extensions.Extension):
    """
    """

    def extendMarkdown(self, md):
        """
        """
        md.registerExtension(self)
        self.processor = BootstrapTreeprocessor()
        self.processor.md = md
        self.processor.config = self.getConfigs()
        md.treeprocessors.add('bootstrap', self.processor, '_end')

s="""|**name**|**age**|
|---|---|
|tim| 16|
|tom| 17|
"""
print(markdown.markdown(s,extensions=['markdown.extensions.toc','markdown.extensions.fenced_code','markdown.extensions.tables',BootStrapExtension()]))

```
输出如下。
```bash
python3 main.py

# 看 class 样式加上去了。
<table class="table table-bordered table-dark">
<thead>
<tr>
<th><strong>name</strong></th>
<th><strong>age</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>tim</td>
<td>16</td>
</tr>
<tr>
<td>tom</td>
<td>17</td>
</tr>
</tbody>
</table>
```
更多关于拓展的内容可以查看 [python-markdown的官方文档](https://python-markdown.github.io/extensions/api/)，上面的编写的这个拓展你也可以在 [github](https://github.com/Neeky/bootstrap-your-markdown) 上找到。

---