## js 中赋值的本质是什么
可以把对象看成是一个盒子，变量名就是粘在盒子上的标签，下面的代码给对象先粘上`x`标签，然后又粘上`y`标签。
```ts
let x = { 'hello': 'world' };
let y = x;

console.log(x, y);
```
运行效果。
```bash
tsc main.ts && node main.js
{ hello: 'world' } { hello: 'world' }
```
也就是说 x 和 y 粘到了同一个盒子上，专业点的话就是说 x 和 y 引用了同一个对象。

![const](static/2020-14/const.png)

google-adsense

---

## 标签可以撕下来
像现实生活一样标签可以从一个盒子上撕下来，粘到另一个盒子上去。下面的代码就把 y 撕下来粘到另一个盒子上去。
```ts
let x = { 'hello': 'world' };
let y = x;

// 把 y 撕下来粘到一个新的盒子(对象)上去

y = { 'name': 'tim' };

console.log(x, y);
```
由于 ts 的检查会报错，所以这里直接以 js 解析。
```bash
node main.ts
{ hello: 'world' } { name: 'tim' }
```
---

同理 x 标签也可以被撕下来。
```ts
let x = { 'hello': 'world' };

// 把 x 撕下来粘到一个新的盒子(对象)上去

x = { 'name': 'tim' };

console.log(x);
```
运行效果。
```bash
node main.ts
{ name: 'tim' }
```

---

## const 声明有什么特点
const 声明粘上去的标签唯一的区别就是不能撕下来，客观上实现了对基本类型的只读。
```ts
const x = "hello world";
x = "hello ts";
console.log(x);
```
编译时会报 x 是常量不能重新分配值，也就是说标签不能斯下来粘到另一个对象上去。
```
main.ts:2:1 - error TS2588: Cannot assign to 'x' because it is a constant.

2 x = "hello ts";
  ~


Found 1 error.
```
---

标签不能撕下来，不是说盒子里的内容不能改。

```ts
const x = { 'hello': 'world' };
console.log(x);
x['foo'] = 'bar';
console.log(x);
```
运行效果。
```bash
tsc main.ts && node main.js
{ hello: 'world' }
{ hello: 'world', foo: 'bar' }
```

---