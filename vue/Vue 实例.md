## 创建一个 Vue 实例
在创建 Vue 实例的时候通过一个“选项对象”的对象来传给 Vue 实例一些基本的信息，比如要挂载到的 DOM 实例，参与到响应式渲染的数据。

![vue instance](static/2020-27/sqlpy-const-floding.jpg)

---


## 数据
在创建 Vue 实例的时候，Vue 会把选项对象中的 `data` 属性指向的字典添加到自己的响应式系统中，当数据发生变化时会补动态的渲染到页面。
```js
data = {
    message: "hello world."
};

const vm = new Vue({
    el: "#app",
    data,
});

data === vm.$data // True
vm.$el === document.getElementById('app') // True
```
data 对象中的所有属性被加到响式系统之后，我不在需要 `vm.$data.message` 来访问 `data` 对象中的值，我们可以直接通过 Vue 实例直接访问。
```js
vm.message
```

![vue-instance](static/2021-01/vue-message.jpeg)

`Object.freeze(obj)` 会阻止修改现有的的属性，如果对选项对象的 `data` 属性应用了 Object.freeze 就等于是响应式系统不再追踪这个对象属性值的变化。

---

## 方法
对于一个 Vue 实例来说除了数据之外剩下的就是方法了，下面我们实现一段逻辑，当 `vm.message` 的值变化后，我们打印一下变化前后的值。
```js
vm.message = "just do it";
```
![vue-instance](static/2021-01/vue-function.jpeg)

---

## Vue 生命周期钩子

Vue 实例有若干个回调钩子，支持我们在其不同的阶段处理一些逻辑。

![vue-hook](https://cn.vuejs.org/images/lifecycle.png)

---