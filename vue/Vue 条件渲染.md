## Vue 条件渲染
Vue 的条件渲染用于控制元素的可见性。

![](static/2021-01/vue-condition-render.jpg)

---

# v-if 
只有当传递给 `v-if` 的值为 `true` 的时候元素才会被渲染。
```html
<div id="app">
    <p v-if="visible">v-if 测试</p>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        visible: true
    } 
});
```
![](static/2021-01/vue-if.jpeg)

可以看到当 `v-if` 的值被设置成 `false` 的时候 p 元素就看不到了;这种看不到不是渲染上的不可见，而是 p 元素被从 dom 中移除了。

---





##  v-else
`v-else` 与 `v-if` 配合使用，功能上与编程语言中的 `if ... else ...` 一样。
```html
<div id="app">
    <p v-if="visible">v-if 测试</p>
    <p v-else>这里是 v-else </p>
</div>
```
![](static/2021-01/vue-else.jpeg)

---

## 分组显示
上面我们用到的 `v-if` 和 `v-else` 只作用于一个元素，如果想同时作用于多个元素，就要用 `template` 标签把它们做成一个组。
```html
<div id="app">
    <template v-if="visible">
        <h2>这里是一组元素</h2>
        <p>hello world.</p>
    </template>
</div>
```
![](static/2021-01/vue-template.jpeg)

---

## 用 KEY 管理可复用元素
Vue 会复用元素以减少重复的渲染，但是被复用元素的属性也被保留下来了，这个有时候就地导致问题。
```html
<div id="app">
    <template v-if="loginType">
        <lable>UserName:</lable>
        <input placeholder="enter your name.">
    </template>
    <template v-else>
        <lable>Email:</lable>
        <input placeholder="enter your email.">
    </template>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        loginType: true
    },
    methods: {
        changeLoginType:function(){
            if(this.loginType == true){
                this.loginType = false;
            }
            else {
                this.loginType = true;
            }
        }
    }
});
```
第一步我们先填写上用户名

![](static/2021-01/vue-key-01.jpeg)

第二步我们再填写邮件

![](static/2021-01/vue-key-02.md.jpg)

这个时候我们可以看到 Vue 由于复用了之前的 `input` 标签，所以之前填写的内容也被带出来了。

解决办法也比较直接，给被重用的元素添加上一个唯一的键就行了，向下面这样。
```html
<div id="app">
    <template v-if="loginType">
        <lable>UserName:</lable>
        <input placeholder="enter your name." key="input-name">
    </template>
    <template v-else>
        <lable>Email:</lable>
        <input placeholder="enter your email." key="input-email">
    </template>
    <button @click="changeLoginType">change login type</button>
</div>
```

---