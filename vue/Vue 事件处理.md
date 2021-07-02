## v-on事件处理
v-on 可以关联事件与事件的处理逻辑，形式上看可以分成三个类型，一、简单的 js 语句 二、函数名 三、函数调用。

![](static/2021-01/simon-fitall-oaLYFQNr19c-unsplash.jpg)

---

## 简单的 js 语句
如果事件对应的逻辑比较简单的话，这个处理方式就是非常优雅的，比如官方给的 `+1` 例子。
```html
<div id="app">
    <p>{{counts}}</p>
    <button @click="counts = counts + 1">加一</button>
</div>
```
`@click` 是 `v-on:click` 的简写(下同)。

---

## 函数名
这种方案有一个好处就是逻辑尽可能的写在了 js 里面。
```html
<div id="app">
    <p>{{counts}}</p>
    <button @click="addOne">加一</button>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        counts:0
    },
    methods: {
        addOne:function() {
            this.counts = this.counts + 1;
        }
    }
});
```
其实吧，在只给事件关联函数名的情况下，Vue 是会在调用函数的时候传一个参数的，这个参数就是触发的事件对象。
```js
const vm = new Vue({
    el: "#app",
    data: {
        counts:0
    },
    methods: {
        addOne:function(...args) {
            console.log("enter function add with args = " + args + " .");
            let object = args[0];
            console.log(object);
            
            for (const key in object) {
                if (Object.hasOwnProperty.call(object, key)) {
                    const element = object[key];
                    
                }
            }
            this.counts = this.counts + 1;
        }
    }
});
```
![](static/2021-01/vue-event-01.jpeg)

---

## 函数名带参数
如果我们给事件关联到的是一个函数调用的话，那么函数的参数在执行的时候就是我们给出的参数，并且不会附加任何其它的参数。
```html
<div id="app">
    <p>{{counts}}</p>
    <button @click="add(2)">加二</button>
</div>
```
```js
const vm = new Vue({
    el: "#app",
    data: {
        counts:0
    },
    methods: {
        add:function(...args) {
            // 由于我们传递了参数所以这个时候 args[0] 就是我们传进来的值
            console.log("enter function with args = " + args);
            console.log("args type = " + typeof(args) + " args length = " + args.length);
            // 因为我们只传了一个参数，所以这个参数可以通过 args[0] 取得
            let detal = args[0];
            this.counts = this.counts + detal ;
        }
    }
});
```
如果是以上面这种函数调用的方式关联事件的话，又想拿到事件，我们可以用 `$event` 来取得事件的引用。
```html
<div id="app">
    <p>{{counts}}</p>
    <button @click="add(2,$event)">加二</button>
</div>
```

---