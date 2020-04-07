## 概要
由于 typescript 有类型声明，所以可以简单实现一些“鸭子类型”的代码。

```ts
function say_hello(HasNameObj: { 'name': string }) {
    console.log(`hello My name is ${HasNameObj.name}.`);
}

let person = { 'name': 'tim','age':16};
say_hello(person);
```

运行效果。

```bash
tsc main.ts && node main.js
hello My name is tim.
```

![interface](static/2020-15/interface.png)

google-adsense

---

## interface 要解决的问题
因为是“鸭子类型”，实际上 person 对象是有 age 属性的，typescript 编译器并不会检查这么多，也就是说多出来的 age 属性并不会使得编译器报错，也就是说编译器做的是最低要求。

如果有多处都要用到这样的“鸭子类型”怎样才能做到不重复？这就要用到 `interface`把公共的部分提取出来。

```ts
interface HasName {
    'name': string,
    'age': number,
}


function sayHello(hasNameObj: HasName) {
    console.log(`hello My name is ${hasNameObj.name}.`);
}

function rawName(hasNameObj: HasName) {
    console.log(hasNameObj.name);
}

let person = { 'name': 'tim', 'age': 16 };
sayHello(person);
rawName(person);
```
运行效果。
```bash
tsc main.ts && node main.js
hello My name is tim.
tim
```

---
