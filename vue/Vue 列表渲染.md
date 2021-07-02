## 列表渲染
Vue 中可以使用 `v-for` 指令来渲染一个列表。
```html
<div id="app">
    <ul>
        <li v-for="person in persons">{{person.name}}  {{person.age}}</li>
    </ul>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        persons:[
            {name:'张三岁',age:18},
            {name:'李四年',age:19},
            {name:'王五载',age:20}
        ]
    },
});
```
![](static/2021-01/kevin-wolf-zPz3OV4MzFE-unsplash.jpg)


---

## v-for 索引解析
`v-for` 不只是能简单的取出列表中的值，稍加修改还可以把索引也带出来。
```html
<div id="app">
    <ul>
        <li v-for="(person,index) in persons">name = {{person.name}} age= {{person.age}} index = {{index}}</li>
    </ul>
</div>
```
同样 `v-for` 也可以用于对象，不过解析的次序分别是 `value,key,index` 这三个维度。
```html
<div id="app">
    <ul>
        <li v-for="(value,key,index) in persons[0]">{{value}} {{key}} {{index}}</li>
    </ul>
</div>
```

---

## 维护状态
当 Vue 正在更新使用 v-for 渲染的元素列表时，它默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。如我们在列表的最前面插入一个元素，就结果来看 Vue 也能正确的处理元素之间的次序。

![](static/2021-01/vue-for-data.jpeg)


---

## v-for 里使用范围

```html
<div id="app">
    <ul>
        <li v-for="number in 5">{{number}}</li>
    </ul>
</div>
```

---


