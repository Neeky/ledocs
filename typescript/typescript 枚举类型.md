## 背景
最近一直在看 [typescript](https://www.typescriptlang.org/docs/home.html) 和 [javascript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide) 的官方文档，看到 typescript 说它可以定义枚举类型，可是 javascript 中并没有这个类型呀。想到 typescript 最终都要转成 javascript 才能运行，突然就非常的好奇，typescript 是怎么做的。

![sqlpy](static/2020-31/sqlpy-enums.jpg)

---

## typescript 如何实现枚举
要想知道 typescript 是怎么做的，一种比较直接的方法就是，定义一个枚举然后看它被转换成怎样的 javascript 代码。
```typescript
enum Direction {
    Up,
    Down,
    Left,
    Right
}

console.log(Direction.Up);
```
转换后的 javascript 代码如下。
```javascript
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 0] = "Up";
    Direction[Direction["Down"] = 1] = "Down";
    Direction[Direction["Left"] = 2] = "Left";
    Direction[Direction["Right"] = 3] = "Right";
})(Direction || (Direction = {}));
console.log(Direction.Up);
```

哈哈哈，原来是这样。

---

## 数值型枚举
从枚举值的类型上来看 MySQL 的枚举类型支持两种类型，第一种是数值，第二种是字符串。经验上看数值型枚举更为常用。
```typescript
enum Direction {
    Up = 100,
    Down = 200,
    Left,
    Right
}
```
数值型枚举会被转换为如下代码。
```js
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 100] = "Up";
    Direction[Direction["Down"] = 200] = "Down";
    Direction[Direction["Left"] = 201] = "Left";
    Direction[Direction["Right"] = 202] = "Right";
})(Direction || (Direction = {}));
```

google-adsense

---

## 字符型枚举
从名字上就可以看出来它的枚举值是字符串。
```typescript
enum Words {
    H = "hello",
    W = "world"
}
```
字符型枚举会被转换为如下代码。
```js
var Words;
(function (Words) {
    Words["H"] = "hello";
    Words["W"] = "world";
})(Words || (Words = {}));
```

---

## 数值枚举的增强
数值枚举作为最常用的枚举类型，typescript 在这个类型上做了增强，使得后面的枚举项可以引用前面枚举项的值。
```ts
enum Nums {
    X = 100,
    Y = 200,
    SUM = X + Y
}
```
转换后的代码如下。
```js
var Nums;
(function (Nums) {
    Nums[Nums["X"] = 100] = "X";
    Nums[Nums["Y"] = 200] = "Y";
    Nums[Nums["SUM"] = 300] = "SUM";
})(Nums || (Nums = {}));
```

---