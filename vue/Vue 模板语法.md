## 模板语法

Vue 使用了一套基于 HTML 的语法，就学习成本来讲还是比较低的；在 Vue 的内部 Vue 会把模板转换成渲染函数，不过刚入门可以先不用管这么多，学好模板怎么用再说。

![vue-template](static/2021-01/camke.jpg)

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
    <p>使用双大括号语法实现插值：&lt;span style='color:red;'&gt;this should be red.&lt;/span&gt;</p>
</div>
```

可以看到当以插值方式渲染值的时候，如果我们的值里面有 html 标签，那么标签都会被转意。那么问题就来了，我们要插入的就是一个 html 子元素的时候应该怎么办呢？

---

## 原始 html

把字符串形式的 html 标签字符串，赋值给 v-html 就能把这段 html 以子元素的形式插入到标签中。
```html
<div id="app" v-html>
    <p>使用双大括号语法实现插值：{{message}}</p>
    <p v-html="message"></p>
</div>
```
```js
data = {
    message: "<span style='color:red;'>this should be red.</span>"
};

const vm = new Vue({
    el: "#app",
    data,
});
```

渲染效果：
![v-html](static/2021-01/vue-v-html-2.jpeg)

从 console 中看到的 html ：

![v-html](static/2021-01/vue-v-html.jpeg)

---

## 元素属性
不管是“插值”还是“原始 html” 解决的都是元素内容的问题，如果我们要设置元素的属性的话，那又是另一个故事了，这个故事主角叫 `v-bind` 。
```html
<div id="app" v-html>
    <p v-bind:id="main_para">第一段。</p>
</div>
```
```js
data = {
    message: "<span style='color:red;'>this should be red.</span>",
    main_para: "ups"
};

const vm = new Vue({
    el: "#app",
    data,
});
```
上面代码我们是想把 p 元素的 id 属性设置为 "ups"，现在我们检查元素来看效果。

![vue-v-bind](static/2021-01/vue-v-bind.jpeg)