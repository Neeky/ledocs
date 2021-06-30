## 计算属性的背景
如果我们要在前端展示一个数组(元素都是整数)的合，使用插值语法可以直接这样写。
```htmll
<div id="app">
    <div>sum ({{numbers}}) = {{numbers.reduce((x,y) => {return x + y;})}} </div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3]
    }
});
```
这样做的话模板里面就有了逻辑，我们把模板做的太“重”了，好的实现方式是把逻辑都放到 js 里。

![](static/2021-01/vue-watch.jpg)

---

## 用函数封装计算逻辑
在 js 中实例求合的逻辑，模板中只做简单的展示。
```html
<div id="app">
    <div>sum ({{numbers}}) = {{sum()}} </div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3]
    },
    methods: {
        sum:function() {
            return this.numbers.reduce((x,y)=>{return x +y;})
        }
    }
});
```
这个解决方案也有一个问题那就是它存在重复计算的问题，这个问题要依靠计算属性才能解决。

---

## 计算属性
计算属性已经是属性了，所以可以省略掉模板里的函数调用。
```html
<div id="app">
    <div >sum ({{numbers}}) = {{cached_sum}} </div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3],
    },
    methods: {
        sum:function() {
            console.log("enter sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    },
    computed: {
        cached_sum:function(){
            console.log("enter cached_sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    }
});

```

---

## 计算属性 vs 函数
如果使用函数对于同一份数据会存在重复计算，也就是说就是数据没有变，函数被调用了几次它就会执行几次。
```html
<div id="app">
    <!-- 循环创建出 4 个  div -->
    <div v-for="element in elements">sum ({{numbers}}) = {{sum()}} </div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3],
        elements:[1,2,3,4]
    },
    methods: {
        sum:function() {
            console.log("enter sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    },
    computed: {
        cached_sum:function(){
            console.log("enter cached_sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    }
});
```

![vue-function](static/2021-01/vue-function-01.jpeg)

可以看在数据变化一次的情况下，由于有 4 处用到了 sum 函数，所以在 numbers 发生改变的时候会有 4 次调整，其实后三次调用没有太大的必要。

为了优化掉额外的三次调用，我们可以使用计算属性。

```html
<div id="app">
    <div v-for="element in elements">sum ({{numbers}}) = {{cached_sum}} </div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3],
        elements:[1,2,3,4]
    },
    methods: {
        sum:function() {
            console.log("enter sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    },
    computed: {
        cached_sum:function(){
            console.log("enter cached_sum function .");
            return this.numbers.reduce((x,y)=>{return x +y;});
        }
    }
});
```

![](static/2021-01/vue-computed.jpeg)


---

## 侦听器
侦听器可以做到当值发生变化后执行特定的业务逻辑。下面实现一个假的需求当 `numbers` 的值发生变化之后，计算一下 `numbers` 中的元素个数。
```html
<div id="app">
    <div> {{numbers}} </div>
    <div>列表里共有 {{counts}} 个元素 .</div>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        numbers:[1,2,3],
        counts: "",
    },
    created:function(){
        // 当 Vue 实例创建完成之后要执行的逻辑
        // 把 counts 由默认的整数设置为 numbers 列表的长度
        this.counts = this.numbers.length;
    },
    watch:{
     numbers:function(newValue,oldValue) {
         console.log("numbers 的值发生变化，之前它是 " + oldValue + " 现在它变成了 " + newValue);
         this.counts = this.numbers.length;
     }   
    }
});
```
![](static/2021-01/vue-watch-01.jpeg)

---