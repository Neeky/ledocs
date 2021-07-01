## Class 与 Style 绑定
由于 class 和 style 是比较常用的属性，Vue 对这两个做了增强，我们先来看一下不做加强时的用法。

![](static/2021-01/vue-style.jpg)

```html
<style>
    .dark {
        /*背景设置为灰色*/
        background-color: darkgray;
    }
    .outline {
        /*边框 4 个像素*/
        border-width: 4px;
    }
</style>

<div id="app">
    <button v-bind:class="cls">{{message}}</button>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        isDark:true,
        isOutline:true,
        message: 'hello world.',
    },
    computed: {
        // 计算 class 属性的值
        cls:function() {
            result = "";
            if(this.isDark == true) {
                result = result + "dark"
            }
            if(this.isOutline == true){
                result = result + " outline"
            }
            return result
        }
    }  
});
```
![](static/2021-01/vue-class-01.jpeg)

可以看到就算没有 class 和 style 的增强，我们自己算出来它们的值也是可以的。

---


## class 增强
1、增强方案一 让 class 支持对象写法，对象的键用于指定 css-class 类名，值的 `true` 或 `false` 决定这个 css 样式是否生效。
```html
<div id="app">
    <button v-bind:class="{dark:isDark,outline:isOutline}">{{message}}</button>
</div>
```

2、第二个更加好理解了，因为 `css` 中 `class` 可以接受多个值,所以我们直接把样子的名字写在数组里面就行了。
```html
<div id="app">
    <button v-bind:class="[dark,outline]">{{message}}</button>
</div>
```
这里有一个要注意的地方因为 `v-bind:class` 用的还是 js 语法，所以变更名之间还是要用 `,` 逗号

---

## style 语法
style 属性和 class 属性一样也支持了对象语法，和数组语法。

---


