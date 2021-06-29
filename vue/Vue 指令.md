## 指令

指令是带有 `v-` 前缀的特殊属性，当指令的值改变时它会将连带的影响更新到 DOM 。 以 `v-if` 为例子，他控制着元素的可见性。

```html
<div id="app">
    <p v-if="visible">{{message}}</p>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        message: "abc",
        visible: true,
    }
});
```

![sqlpy](static/2020-22/sqlpy-0610-npm.jpg)

默认情况下 app 这个元素是可见的，当我们把 `v-if` 的值 `visible` 由 `true` 改成 `false` 之后整个元素就不可见了。

![vue-direction](static/2021-01/vue-direction-01.jpeg)

更新 `v-if` 的值为 `false` 之后，元素就不可见了。

![vue-direction](static/2021-01/vue-direction-02.jpeg)

---


## 指令参数
之前我们使用指令的方式是 `<指令名>=<指令值>` ，对于指令来说还有一个重要的部分它就是指令参数。`<指令名>:[指令参数]=<指令值>` ，例如我们想设置 a 标签的 href 属性
可以这样做。
```html
<div id="app">
    <a v-bind:href="web_site">go to baidu</a>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        web_site: "https://www.baidu.com"
    }
});
```
![vue-direction](static/2021-01/vue-direction-03.jpeg)


---

## 修饰符
修饰符用 "." 号表示，指明特殊后缀，用于指明指令应该以特殊方式绑定，如 `prevent` 可用用于阻止元素的默认行为。
```html
<div id="app">
    <a v-bind:href="web_site" v-on:click.prevent="showalert">go to baidu</a>
</div>
```
不要再跳转，取而代之的是给出一个提示。
```js
const vm = new Vue({
    el: "#app",
    data: {
        web_site: "https://www.baidu.com"
    },
    methods: {
        showalert:function() {
            alert("不要跳到 baidu 去了。");
        }
    }
});
```
现在点击页面上的超链接不再会跳转，而是会弹出一个提示。
![vue-direction](static/2021-01/vue-direction-04.jpeg)

---

## 缩写
对于非常常用的 `v-bind` 和 `v-on` 指令，Vue 为他们提供了缩写

|**原始指令**|**缩写指令**|
|-----------|-----------|
|`v-bind[参数]=[值]`   | `:[参数]=[值]` |
|`v-on[参数]=[值]`     | `@[参数]=[值]` |


---