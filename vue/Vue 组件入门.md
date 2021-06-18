## Vue-组件是什么
在 Web 开发总有一些东西是重复的，比如列表中的元素，表格中的行；Vue 中组件把这些重复的样式和行为给抽象出来，光是这么说还是不太好理解，下面看一个个的例子吧。
![sqlpy](static/2020-25/sqlpy-connection.jpg)

---

## 组件入门一
假设列表中的每一项都是 "hello world." ，那么列表代码看起来就是这样的。
```html
<ul>
    <li>hello world.</li>
    <li>hello world.</li>
    <li>hello wrold.</li>
</ul>
```
上面的这个例子中所有的值都是死的，没有任何可变的部分，也就是说没有抽象共性的必要；如果硬是要用组件化技术重写也不是不可以。

---

向 Vue 注册一个只显示 `hello world.` 的组件 `message-item`，
```js
Vue.component('message-item',{
    template:"<li>hello world.</li>",
});
```
介于我刚才实现了一个如此有用的组件，那么不在 html 页面里用一下，简直就是没人性呀。
```html
<div id="app">
    <ul>
        <message-item></message-item>
        <message-item></message-item>
        <message-item></message-item>
    </ul>
</div>
```
初始化 Vue 实例。
```js
var app = new Vue({
    el: '#app',
});
```

页面的完整代码如下：
```html
<html>
    <header>
        <title>Vue</title>
    </header>

    <body>
        <ul>
            <li>hello world.</li>
            <li>hello world.</li>
            <li>hello wrold.</li>
        </ul>

        <hr>

        <div id="app">
            <ul>
                <message-item></message-item>
                <message-item></message-item>
                <message-item></message-item>
            </ul>
        </div>
    </body>

    <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
    <script>
        Vue.component('message-item',{
            template:"<li>hello world.</li>",
        });

        var app = new Vue({
            el: '#app',
        });
    </script>
</html>
```


![sqlpy](static/2021-01/vue-component-01.jpeg)

---

## 组件入门二
上一个例子中我们的组件是“死”的，比如要实现三个不同的列表项 html 可以这样
```html
<ul>
    <li>hello js.</li>
    <li>hello python.</li>
    <li>hello java.</li>
</ul>
```
但是在不改动 `message-item` 组件的情况下是实现不了的(原因在于 message-item 已经被写死了。)；为了让 li 中的内容不固定，我们可以让 message-item 支持传参，传递进来的参数就是要显示的内容。
```html
Vue.component('message-item',{
    template:"<li>{{message}}</li>", // 展示 message 变量的值
    props:['message']                // 定义一个 message 属性，
});
```
完整的代码就成了这样。
```html
<html>
    <header>
        <title>Vue</title>
    </header>

    <body>
        <ul>
            <li>hello js.</li>
            <li>hello python.</li>
            <li>hello java.</li>
        </ul>

        <hr>

        <div id="app">
            <ul>
                <message-item message="hello js."></message-item>
                <message-item message="hello python."></message-item>
                <message-item message="hello java."></message-item>
            </ul>
        </div>
    </body>

    <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
    <script>
        Vue.component('message-item',{
            template:"<li>{{message}}</li>", // 展示 message 变量的值
            props:['message']                // 定义一个 message 属性，
        });

        var app = new Vue({
            el: '#app',
        });
    </script>
</html>
```
![sqlpy](static/2021-01/vue-component-02.jpeg)

---

## 组件入门三
之前我们说 Vue 组件的这一层抽象是为了消除重复，但是我们在 html 里面对于每一个列表项还是创建了一个`message-item`标签，我们可以借助 v-for 指令把这个也消除掉。
```html
<html>
    <header>
        <title>Vue</title>
    </header>

    <body>
        <ul>
            <li>hello js.</li>
            <li>hello python.</li>
            <li>hello java.</li>
        </ul>

        <hr>

        <div id="app">
            <ul>
                <message-item v-bind:message="message" v-for="message in messages"></message-item>
            </ul>
        </div>
    </body>

    <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
    <script>
        Vue.component('message-item',{
            template:"<li>{{message}}</li>", // 展示 message 变量的值
            props:['message']     // 定义一个 message 属性，
        });

        var app = new Vue({
            el: '#app',
            data:{
                messages:['hello js.','hello python.','hello java.']
        }
        });
    </script>
</html>
```
![sqlpy](static/2021-01/vue-component-02.jpeg)

---

好吧，到这里就把列表项的显示部分都抽象到了 message-item 组件里了，今天就悟到了这么多。

---
