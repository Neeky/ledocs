## 模板语法

Vue 使用了一套基于 HTML 的语法，就学习成本来讲还是比较低的；在 Vue 的内部 Vue 会把模板转换成渲染函数，不过刚入门可以先不用管这么多，学好模板怎么用再说。

---

## 插值
和其它的模板系统类似，Vue 的模板系统也是使用双大括号 `{{}}` 来做文本插值。
```html
<div id="app">
    <p>使用双大括号语法实现插值：{{message}}</p>
</div>
``` 
渲染的时候双大括号标签占位点会被 `message` 的值取代，渲染之后的 html 如下
```html
<div id="app">
    <p>使用双大括号语法实现插值：<span style='color:red;'>this should be red.</span></p>
</div>
```
刚才我们在 p 标签内插入了一段文本，如果想插入一段 html 的话，要用 v-html 指令才行。

---

## 原始 html


---