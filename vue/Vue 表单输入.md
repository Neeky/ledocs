## 表单输入

`v-model` 本质上是语法糖，它负责监听用户输入并更新数据，从而达到数据双向绑定的效果。

![](static/2021-01/robert-v-ruggiero-w6VGdenFu3c-unsplash.jpg)

---

## 文本
```html
<div id="app">
    <!-- 变量给到 v-model 这样双向绑定就完成了-->
    <input type="text" v-model="message">
    <p>message = {{message}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        message:"hello world."
    }
});
```
![](static/2021-01/vue-model-input-text.jpg)

---

## 多行文本
这情况下没有 input 标签可用了，只能用 textarea，不过 `v-model` 的语法还是保持了一致。
```html
<div id="app">
    <textarea v-model="message"></textarea>
    <p>message = {{message}}</p>
</div>
```

![](static/2021-01/vue-model-textarea.jpg)

---

## 复选框
对于单个复选框来说绑定到一个 bool 值来代表是否选中是比较合适的，如果是多个复选框我们可以绑定到一个数组来解决问题。
```html
<div id="app">
    <input type="checkbox" v-model="ischecked" value="设置了 v-model value 就是无效的初始值">
    <p>ischecked = {{ischecked}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        ischecked:false
    }
});
```
![](static/2021-01/vue-checkbox-01.jpg)

---
```html
<div id="app">
    <input type="checkbox" v-model="persons" value="张三岁">
    <input type="checkbox" v-model="persons" value="李四年">
    <input type="checkbox" v-model="persons" value="王五载">
    <p>persons = {{persons}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        ischecked:false,
        persons:["张三岁"],
    }
});
```

![](static/2021-01/vue-checkbox-2.jpg)

---


## 单选框
```html
<div id="app">
    <p>给他一亿:</p>
    <input type="radio" v-model="name" value="王多鱼" id="id1"><span>王多鱼</span><br>
    <input type="radio" v-model="name" value="王二小" id="id2"><span>王二小</span><br>
    <p>被选中的人： {{name}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        ischecked:false,
        name:"王多鱼"
    }
});
```
![](static/2021-01/vue-radio-01.jpg)

---


## 选择框
选择框默认也是单选的。
```html
<div id="app">
    <p>给他一亿:</p>
    <select v-model="name">
        <option disabled value="">请选择</option>
        <option value="多鱼">王多鱼</option>
        <option value="二小">王二小</option>
    </select>
    <p>被选中的人： {{name}}</p>
</div>
```
![](static/2021-01/vue-select-single.jpg)

---

通过修改 html 属性可以把选择框变成多选的，这种结构下被选择的值保存在数组中是最合适的。
```html
<div id="app">
    <p>给他一亿:</p>
    <select v-model="name" multiple>
        <option disabled value="">请选择</option>
        <option value="多鱼">王多鱼</option>
        <option value="二小">王二小</option>
    </select>
    <p>被选中的人： {{name}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        ischecked:false,
        name:"" // 虽然这里设置成 [] 数组最为合适，我把默认值设置为 “” 是为了说明 vue 在我们选择完成之后自动的设置成合适的数据类型
    }
});
```
![](static/2021-01/vue-select-multi.jpg)

---

可以看到前面的例子中我们重复的写了多次的 `option` 借助 `v-for` & `v-bind` 我们可以把这些重复的内容消去。
```html
<div id="app">
    <p>给他一亿:</p>
    <select v-model="picked" multiple>
        <option disabled value="">请选择</option>
        <option v-for="name in names" v-bind:value="name">{{name}}</option>
    </select>
    <p>被选中的人： {{picked}}</p>
</div>
```
```js
const vm = new Vue({
    el:"#app",
    data:{
        ischecked:false,
        names:["张三岁","李四年","王五栽"],
        picked:"",
    }
});
```
![](static/2021-01/vue-option.jpg)

---

## 修饰符
`.number` 可以帮我们完成自动的转型
```html
<div id="app">
    今天已经 <input type="text" v-model.number="age"> 岁啦!
</div>
```
![](static/2021-01/vue-number.jpg)

`.trim` 可以帮我们自动去年首尾两端的空格。
```html
<div id="app">
    你的名字 <input type="text" v-model.trim="name">
    <p>Vue 会帮我们自动去空格 '{{name}}'</p>
</div>
```

![](static/2021-01/vue-trim.jpg)

---